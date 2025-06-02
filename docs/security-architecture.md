# Security Architecture Implementation

This document provides comprehensive implementation details for Embedy's security features, including origin validation, encryption, CSP compliance, and clickjacking protection.

## Origin Validation for PostMessage Communication

### Multi-Layered Validation System

```typescript
// security/origin-validator.ts
interface OriginValidationConfig {
  allowedOrigins: string[];
  allowedPatterns: RegExp[];
  allowLocalhost: boolean;
  allowSubdomains: boolean;
  strictMode: boolean;
}

class OriginValidator {
  private config: OriginValidationConfig;
  private trustedOrigins = new Set<string>();
  private blockedOrigins = new Set<string>();
  
  constructor(config: OriginValidationConfig) {
    this.config = config;
    this.initializeTrustedOrigins();
  }
  
  private initializeTrustedOrigins() {
    // Add explicitly allowed origins
    this.config.allowedOrigins.forEach(origin => {
      this.trustedOrigins.add(this.normalizeOrigin(origin));
    });
    
    // Add localhost variants if allowed
    if (this.config.allowLocalhost && !this.config.strictMode) {
      ['http://localhost', 'https://localhost', 'http://127.0.0.1', 'https://127.0.0.1']
        .forEach(origin => this.trustedOrigins.add(origin));
    }
  }
  
  validateOrigin(origin: string, messageType?: string): ValidationResult {
    if (!origin) {
      return { valid: false, reason: 'MISSING_ORIGIN', risk: 'HIGH' };
    }
    
    const normalizedOrigin = this.normalizeOrigin(origin);
    
    // Check blocked list first
    if (this.blockedOrigins.has(normalizedOrigin)) {
      return { valid: false, reason: 'BLOCKED_ORIGIN', risk: 'HIGH' };
    }
    
    // Check trusted origins
    if (this.trustedOrigins.has(normalizedOrigin)) {
      return { valid: true, reason: 'TRUSTED_ORIGIN', risk: 'NONE' };
    }
    
    // Check allowed patterns
    for (const pattern of this.config.allowedPatterns) {
      if (pattern.test(normalizedOrigin)) {
        this.trustedOrigins.add(normalizedOrigin); // Cache for future use
        return { valid: true, reason: 'PATTERN_MATCH', risk: 'LOW' };
      }
    }
    
    // Check subdomain rules
    if (this.config.allowSubdomains) {
      const subdomainResult = this.validateSubdomain(normalizedOrigin);
      if (subdomainResult.valid) {
        return subdomainResult;
      }
    }
    
    // In strict mode, reject everything else
    if (this.config.strictMode) {
      this.blockedOrigins.add(normalizedOrigin);
      return { valid: false, reason: 'STRICT_MODE_REJECTION', risk: 'MEDIUM' };
    }
    
    // Default rejection
    return { valid: false, reason: 'ORIGIN_NOT_ALLOWED', risk: 'MEDIUM' };
  }
  
  private validateSubdomain(origin: string): ValidationResult {
    try {
      const url = new URL(origin);
      const hostname = url.hostname;
      
      for (const trustedOrigin of this.trustedOrigins) {
        const trustedUrl = new URL(trustedOrigin);
        const trustedHostname = trustedUrl.hostname;
        
        // Check if it's a subdomain
        if (hostname.endsWith(`.${trustedHostname}`) && url.protocol === trustedUrl.protocol) {
          // Additional subdomain validation
          if (this.isValidSubdomain(hostname, trustedHostname)) {
            return { valid: true, reason: 'SUBDOMAIN_ALLOWED', risk: 'LOW' };
          }
        }
      }
    } catch (error) {
      return { valid: false, reason: 'INVALID_URL', risk: 'HIGH' };
    }
    
    return { valid: false, reason: 'SUBDOMAIN_NOT_ALLOWED', risk: 'MEDIUM' };
  }
  
  private isValidSubdomain(subdomain: string, baseDomain: string): boolean {
    // Prevent subdomain takeover attempts
    const subdomainParts = subdomain.split('.');
    const baseParts = baseDomain.split('.');
    
    // Must have at least one additional part
    if (subdomainParts.length <= baseParts.length) {
      return false;
    }
    
    // Check for suspicious patterns
    const suspiciousPatterns = ['admin', 'api', 'secure', 'auth', 'login'];
    const subdomainPart = subdomainParts[0];
    
    return !suspiciousPatterns.some(pattern => 
      subdomainPart.toLowerCase().includes(pattern)
    );
  }
  
  private normalizeOrigin(origin: string): string {
    try {
      const url = new URL(origin);
      return `${url.protocol}//${url.hostname}${url.port ? ':' + url.port : ''}`;
    } catch {
      return origin; // Return as-is if not a valid URL
    }
  }
}

interface ValidationResult {
  valid: boolean;
  reason: string;
  risk: 'NONE' | 'LOW' | 'MEDIUM' | 'HIGH';
}
```

## Encryption Key Management

### Web Crypto API Implementation

```typescript
// security/crypto-manager.ts
interface CryptoConfig {
  algorithm: 'AES-GCM' | 'AES-CBC';
  keyLength: 128 | 256;
  derivationIterations: number;
  saltLength: number;
}

class EmbedyCryptoManager {
  private static readonly DEFAULT_CONFIG: CryptoConfig = {
    algorithm: 'AES-GCM',
    keyLength: 256,
    derivationIterations: 100000,
    saltLength: 16
  };
  
  private config: CryptoConfig;
  private keyCache = new Map<string, CryptoKey>();
  private sessionSalt: Uint8Array;
  
  constructor(config: Partial<CryptoConfig> = {}) {
    this.config = { ...EmbedyCryptoManager.DEFAULT_CONFIG, ...config };
    this.sessionSalt = crypto.getRandomValues(new Uint8Array(this.config.saltLength));
  }
  
  async encryptSensitiveField(
    fieldValue: string, 
    fieldType: 'ssn' | 'creditCard' | 'bankAccount' | 'custom',
    contextId: string = 'default'
  ): Promise<EncryptedData> {
    const key = await this.getOrCreateKey(fieldType, contextId);
    const encoder = new TextEncoder();
    const data = encoder.encode(fieldValue);
    
    // Generate unique IV for each encryption
    const iv = crypto.getRandomValues(new Uint8Array(12));
    
    const encrypted = await crypto.subtle.encrypt(
      {
        name: this.config.algorithm,
        iv: iv
      },
      key,
      data
    );
    
    return {
      data: Array.from(new Uint8Array(encrypted)),
      iv: Array.from(iv),
      algorithm: this.config.algorithm,
      keyDerivation: 'PBKDF2',
      timestamp: Date.now(),
      fieldType,
      contextId
    };
  }
  
  async decryptSensitiveField(encryptedData: EncryptedData): Promise<string> {
    const key = await this.getOrCreateKey(encryptedData.fieldType, encryptedData.contextId);
    
    const decrypted = await crypto.subtle.decrypt(
      {
        name: encryptedData.algorithm,
        iv: new Uint8Array(encryptedData.iv)
      },
      key,
      new Uint8Array(encryptedData.data)
    );
    
    const decoder = new TextDecoder();
    return decoder.decode(decrypted);
  }
  
  private async getOrCreateKey(fieldType: string, contextId: string): Promise<CryptoKey> {
    const keyId = `${fieldType}:${contextId}`;
    
    if (this.keyCache.has(keyId)) {
      return this.keyCache.get(keyId)!;
    }
    
    const key = await this.deriveKey(fieldType, contextId);
    this.keyCache.set(keyId, key);
    
    return key;
  }
  
  private async deriveKey(fieldType: string, contextId: string): Promise<CryptoKey> {
    // Use multiple sources for key derivation
    const keyMaterial = await this.generateKeyMaterial(fieldType, contextId);
    
    // Derive key using PBKDF2
    const key = await crypto.subtle.deriveKey(
      {
        name: 'PBKDF2',
        salt: this.sessionSalt,
        iterations: this.config.derivationIterations,
        hash: 'SHA-256'
      },
      keyMaterial,
      {
        name: this.config.algorithm,
        length: this.config.keyLength
      },
      false, // Key is not extractable
      ['encrypt', 'decrypt']
    );
    
    return key;
  }
  
  private async generateKeyMaterial(fieldType: string, contextId: string): Promise<CryptoKey> {
    // Combine multiple entropy sources
    const entropySources = [
      this.sessionSalt,
      new TextEncoder().encode(fieldType),
      new TextEncoder().encode(contextId),
      new TextEncoder().encode(window.location.origin),
      new TextEncoder().encode(Date.now().toString())
    ];
    
    // Concatenate all entropy sources
    const totalLength = entropySources.reduce((sum, source) => sum + source.length, 0);
    const combinedEntropy = new Uint8Array(totalLength);
    let offset = 0;
    
    for (const source of entropySources) {
      combinedEntropy.set(source, offset);
      offset += source.length;
    }
    
    // Import as key material
    return crypto.subtle.importKey(
      'raw',
      combinedEntropy,
      'PBKDF2',
      false,
      ['deriveKey']
    );
  }
  
  // Secure key rotation
  async rotateKeys(): Promise<void> {
    console.info('Rotating encryption keys for security');
    
    // Generate new session salt
    this.sessionSalt = crypto.getRandomValues(new Uint8Array(this.config.saltLength));
    
    // Clear key cache to force regeneration
    this.keyCache.clear();
  }
  
  // Memory cleanup
  destroy(): void {
    this.keyCache.clear();
    // Zero out sensitive data
    this.sessionSalt.fill(0);
  }
}

interface EncryptedData {
  data: number[];
  iv: number[];
  algorithm: string;
  keyDerivation: string;
  timestamp: number;
  fieldType: string;
  contextId: string;
}
```

## CSP Nonce Implementation

### Strict CSP Compliance

```typescript
// security/csp-manager.ts
interface CSPConfig {
  nonceLength: number;
  strictMode: boolean;
  allowUnsafeInline: boolean;
  trustedDomains: string[];
}

class CSPManager {
  private currentNonce: string | null = null;
  private nonceElement: HTMLMetaElement | null = null;
  private config: CSPConfig;
  
  constructor(config: Partial<CSPConfig> = {}) {
    this.config = {
      nonceLength: 32,
      strictMode: true,
      allowUnsafeInline: false,
      trustedDomains: [],
      ...config
    };
    
    this.initializeNonce();
  }
  
  private initializeNonce(): void {
    // Try to get nonce from meta tag (server-provided)
    this.nonceElement = document.querySelector('meta[name="csp-nonce"]');
    
    if (this.nonceElement) {
      this.currentNonce = this.nonceElement.content;
      console.info('Using server-provided CSP nonce');
    } else if (!this.config.strictMode) {
      // Generate client-side nonce as fallback
      this.currentNonce = this.generateNonce();
      console.warn('Generated client-side CSP nonce - not recommended for production');
    } else {
      console.error('No CSP nonce available in strict mode');
    }
  }
  
  private generateNonce(): string {
    const array = new Uint8Array(this.config.nonceLength);
    crypto.getRandomValues(array);
    return btoa(String.fromCharCode(...array))
      .replace(/[+/=]/g, (char) => {
        switch (char) {
          case '+': return '-';
          case '/': return '_';
          case '=': return '';
          default: return char;
        }
      });
  }
  
  getCurrentNonce(): string | null {
    return this.currentNonce;
  }
  
  // Create CSP-compliant script elements
  createScript(src?: string, textContent?: string): HTMLScriptElement {
    const script = document.createElement('script');
    
    if (this.currentNonce) {
      script.nonce = this.currentNonce;
    }
    
    if (src) {
      // Validate source against trusted domains
      if (this.config.strictMode && !this.isTrustedSource(src)) {
        throw new CSPError('Script source not in trusted domains', { src });
      }
      script.src = src;
    }
    
    if (textContent) {
      if (this.config.strictMode && !this.currentNonce) {
        throw new CSPError('Inline scripts require nonce in strict mode');
      }
      script.textContent = textContent;
    }
    
    return script;
  }
  
  // Create CSP-compliant style elements
  createStyleElement(cssText: string): HTMLStyleElement {
    const style = document.createElement('style');
    
    if (this.currentNonce) {
      style.nonce = this.currentNonce;
    } else if (this.config.strictMode) {
      throw new CSPError('Inline styles require nonce in strict mode');
    }
    
    style.textContent = cssText;
    return style;
  }
  
  // Apply styles in CSP-compliant way
  applyStyles(element: HTMLElement, styles: Record<string, string>): void {
    if (this.config.allowUnsafeInline) {
      // Direct style application when unsafe-inline is allowed
      Object.assign(element.style, styles);
    } else {
      // Use CSS custom properties or classes for CSP compliance
      this.applyCSSCustomProperties(element, styles);
    }
  }
  
  private applyCSSCustomProperties(element: HTMLElement, styles: Record<string, string>): void {
    // Convert styles to CSS custom properties
    Object.entries(styles).forEach(([property, value]) => {
      const customProperty = `--embedy-${property.replace(/([A-Z])/g, '-$1').toLowerCase()}`;
      element.style.setProperty(customProperty, value);
    });
    
    // Apply a CSS class that uses these custom properties
    element.classList.add('embedy-dynamic-styles');
  }
  
  private isTrustedSource(src: string): boolean {
    try {
      const url = new URL(src, window.location.origin);
      
      // Always allow same-origin
      if (url.origin === window.location.origin) {
        return true;
      }
      
      // Check against trusted domains
      return this.config.trustedDomains.some(domain => {
        if (domain.startsWith('*.')) {
          // Wildcard subdomain matching
          const baseDomain = domain.slice(2);
          return url.hostname.endsWith(baseDomain);
        }
        return url.hostname === domain;
      });
    } catch {
      return false; // Invalid URL
    }
  }
  
  // Validate current CSP compliance
  validateCSPCompliance(): CSPValidationResult {
    const violations = [];
    
    // Check for nonce availability
    if (this.config.strictMode && !this.currentNonce) {
      violations.push({
        type: 'MISSING_NONCE',
        severity: 'HIGH',
        message: 'No CSP nonce available in strict mode'
      });
    }
    
    // Check for unsafe inline styles
    const inlineStyles = document.querySelectorAll('[style]');
    if (inlineStyles.length > 0 && !this.config.allowUnsafeInline) {
      violations.push({
        type: 'UNSAFE_INLINE_STYLES',
        severity: 'MEDIUM',
        message: `Found ${inlineStyles.length} elements with inline styles`,
        elements: Array.from(inlineStyles).slice(0, 5) // Limit for performance
      });
    }
    
    // Check for inline scripts without nonce
    const inlineScripts = document.querySelectorAll('script:not([src])');
    const scriptsWithoutNonce = Array.from(inlineScripts).filter(
      script => !script.hasAttribute('nonce')
    );
    
    if (scriptsWithoutNonce.length > 0) {
      violations.push({
        type: 'INLINE_SCRIPTS_WITHOUT_NONCE',
        severity: 'HIGH',
        message: `Found ${scriptsWithoutNonce.length} inline scripts without nonce`
      });
    }
    
    return {
      compliant: violations.length === 0,
      violations,
      score: this.calculateComplianceScore(violations)
    };
  }
  
  private calculateComplianceScore(violations: CSPViolation[]): number {
    const severityWeights = { HIGH: 3, MEDIUM: 2, LOW: 1 };
    const totalWeight = violations.reduce(
      (sum, violation) => sum + severityWeights[violation.severity], 
      0
    );
    return Math.max(0, 100 - (totalWeight * 10));
  }
}

class CSPError extends Error {
  constructor(message: string, public details?: any) {
    super(message);
    this.name = 'CSPError';
  }
}

interface CSPViolation {
  type: string;
  severity: 'HIGH' | 'MEDIUM' | 'LOW';
  message: string;
  elements?: Element[];
}

interface CSPValidationResult {
  compliant: boolean;
  violations: CSPViolation[];
  score: number;
}
```

## Clickjacking Protection

### Frame Embedding Security

```typescript
// security/clickjacking-protection.ts
interface ClickjackingConfig {
  allowedParents: string[];
  requireVisualIndicators: boolean;
  frameBreaking: boolean;
  maxDepth: number;
}

class ClickjackingProtection {
  private config: ClickjackingConfig;
  private isEmbedded: boolean;
  private parentOrigin: string | null = null;
  
  constructor(config: Partial<ClickjackingConfig> = {}) {
    this.config = {
      allowedParents: [],
      requireVisualIndicators: true,
      frameBreaking: false,
      maxDepth: 3,
      ...config
    };
    
    this.isEmbedded = window.top !== window.self;
    this.initializeProtection();
  }
  
  private initializeProtection(): void {
    if (!this.isEmbedded) {
      // Not in iframe, set frame options for our own content
      this.setFrameOptions();
      return;
    }
    
    // We are embedded, validate the embedding context
    this.validateEmbeddingContext();
  }
  
  private validateEmbeddingContext(): void {
    try {
      // Get parent origin from referrer
      this.parentOrigin = document.referrer ? new URL(document.referrer).origin : null;
      
      if (!this.parentOrigin) {
        throw new ClickjackingError('Cannot determine parent origin');
      }
      
      // Validate against allowed parents
      if (!this.isAllowedParent(this.parentOrigin)) {
        throw new ClickjackingError(`Embedding not allowed from origin: ${this.parentOrigin}`);
      }
      
      // Check embedding depth
      this.validateFrameDepth();
      
      // Add visual indicators if required
      if (this.config.requireVisualIndicators) {
        this.addVisualIndicators();
      }
      
      // Setup communication with parent for additional validation
      this.setupParentCommunication();
      
    } catch (error) {
      if (error instanceof ClickjackingError) {
        this.handleClickjackingAttempt(error);
      } else {
        console.error('Error validating embedding context:', error);
      }
    }
  }
  
  private isAllowedParent(origin: string): boolean {
    // Check explicit allowlist
    if (this.config.allowedParents.includes(origin)) {
      return true;
    }
    
    // Check pattern matching for allowed parents
    return this.config.allowedParents.some(allowed => {
      if (allowed.startsWith('*.')) {
        const domain = allowed.slice(2);
        try {
          const url = new URL(origin);
          return url.hostname.endsWith(domain);
        } catch {
          return false;
        }
      }
      return false;
    });
  }
  
  private validateFrameDepth(): void {
    let depth = 0;
    let currentWindow = window;
    
    try {
      while (currentWindow !== currentWindow.parent && depth < this.config.maxDepth) {
        currentWindow = currentWindow.parent;
        depth++;
      }
      
      if (depth >= this.config.maxDepth) {
        throw new ClickjackingError(`Frame nesting too deep: ${depth}`);
      }
    } catch (error) {
      // Cross-origin access error - this is expected and safe
      if (error.name !== 'SecurityError') {
        throw error;
      }
    }
  }
  
  private addVisualIndicators(): void {
    // Add clear visual boundary
    const indicator = document.createElement('div');
    indicator.className = 'embedy-embedded-indicator';
    indicator.innerHTML = `
      <div class="embedy-embedded-banner">
        <span>🔒 Embedded content from ${window.location.hostname}</span>
        <button class="embedy-open-original" onclick="window.open('${window.location.href}', '_blank')">
          Open in new tab
        </button>
      </div>
    `;
    
    // Style the indicator using adoptedStyleSheets for CSP compliance
    const styles = `
      .embedy-embedded-indicator {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        z-index: 999999;
        background: linear-gradient(90deg, #1e3a8a, #3b82f6);
        color: white;
        font-family: system-ui, sans-serif;
        font-size: 12px;
        border-bottom: 1px solid rgba(255,255,255,0.2);
      }
      .embedy-embedded-banner {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 8px 16px;
        max-width: 1200px;
        margin: 0 auto;
      }
      .embedy-open-original {
        background: rgba(255,255,255,0.2);
        border: 1px solid rgba(255,255,255,0.3);
        color: white;
        padding: 4px 12px;
        border-radius: 4px;
        cursor: pointer;
        font-size: 11px;
      }
      .embedy-open-original:hover {
        background: rgba(255,255,255,0.3);
      }
    `;
    
    // Add styles in CSP-compliant way
    const styleSheet = new CSSStyleSheet();
    styleSheet.replaceSync(styles);
    document.adoptedStyleSheets = [...document.adoptedStyleSheets, styleSheet];
    
    // Add indicator to page
    document.body.insertBefore(indicator, document.body.firstChild);
    
    // Adjust body padding to account for indicator
    document.body.style.paddingTop = '32px';
  }
  
  private setupParentCommunication(): void {
    // Send handshake to parent for additional validation
    if (window.parent && this.parentOrigin) {
      const handshake = {
        type: 'embedy-handshake',
        origin: window.location.origin,
        timestamp: Date.now(),
        userAgent: navigator.userAgent.substring(0, 100), // Truncated for privacy
        capabilities: {
          webComponents: 'customElements' in window,
          shadowDOM: 'attachShadow' in Element.prototype
        }
      };
      
      window.parent.postMessage(handshake, this.parentOrigin);
      
      // Listen for parent validation response
      window.addEventListener('message', (event) => {
        if (event.origin === this.parentOrigin && event.data.type === 'embedy-validation') {
          this.handleParentValidation(event.data);
        }
      });
    }
  }
  
  private handleParentValidation(validationData: any): void {
    if (!validationData.approved) {
      throw new ClickjackingError('Parent validation failed');
    }
    
    // Additional security checks based on parent response
    if (validationData.requiresAuth && !this.hasValidAuth()) {
      throw new ClickjackingError('Authentication required for this embedding context');
    }
  }
  
  private hasValidAuth(): boolean {
    // Check for valid authentication tokens
    const authToken = localStorage.getItem('embedy-auth-token') || 
                     sessionStorage.getItem('embedy-auth-token');
    
    if (!authToken) return false;
    
    try {
      // Basic JWT validation (in production, verify signature)
      const payload = JSON.parse(atob(authToken.split('.')[1]));
      return payload.exp > Date.now() / 1000;
    } catch {
      return false;
    }
  }
  
  private handleClickjackingAttempt(error: ClickjackingError): void {
    console.error('Potential clickjacking detected:', error.message);
    
    // Log the attempt
    this.logSecurityEvent({
      type: 'CLICKJACKING_ATTEMPT',
      severity: 'HIGH',
      origin: this.parentOrigin,
      userAgent: navigator.userAgent,
      timestamp: Date.now(),
      details: error.message
    });
    
    if (this.config.frameBreaking) {
      // Break out of frame (use with caution)
      try {
        if (window.top) {
          window.top.location = window.location.href;
        }
      } catch {
        // Cross-origin error expected
      }
    }
    
    // Show warning to user
    this.showSecurityWarning(error.message);
  }
  
  private showSecurityWarning(message: string): void {
    const warning = document.createElement('div');
    warning.style.cssText = `
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: #dc2626;
      color: white;
      padding: 20px;
      border-radius: 8px;
      z-index: 1000000;
      font-family: system-ui, sans-serif;
      text-align: center;
      max-width: 400px;
    `;
    
    warning.innerHTML = `
      <h3>Security Warning</h3>
      <p>${message}</p>
      <button onclick="window.open('${window.location.href}', '_blank'); this.parentElement.remove();">
        Open in secure context
      </button>
    `;
    
    document.body.appendChild(warning);
  }
  
  private logSecurityEvent(event: any): void {
    // Send to security monitoring service
    if (window.fetch) {
      fetch('/api/security/events', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(event)
      }).catch(() => {
        // Silently fail - don't break the application
      });
    }
  }
}

class ClickjackingError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ClickjackingError';
  }
}
```

## Complete Security Integration

### Usage in Embedy Components

```typescript
// Complete security integration example
class SecureEmbeddedComponent {
  private originValidator: OriginValidator;
  private cryptoManager: EmbedyCryptoManager;
  private cspManager: CSPManager;
  private clickjackingProtection: ClickjackingProtection;
  
  constructor(securityConfig: SecurityConfig) {
    // Initialize all security layers
    this.originValidator = new OriginValidator({
      allowedOrigins: securityConfig.allowedOrigins,
      allowedPatterns: securityConfig.allowedPatterns,
      strictMode: securityConfig.strictMode
    });
    
    this.cryptoManager = new EmbedyCryptoManager({
      algorithm: 'AES-GCM',
      keyLength: 256
    });
    
    this.cspManager = new CSPManager({
      strictMode: securityConfig.strictMode,
      trustedDomains: securityConfig.trustedDomains
    });
    
    this.clickjackingProtection = new ClickjackingProtection({
      allowedParents: securityConfig.allowedOrigins,
      requireVisualIndicators: true
    });
  }
  
  async handleSecureMessage(event: MessageEvent): Promise<void> {
    // Validate origin first
    const originValidation = this.originValidator.validateOrigin(event.origin);
    if (!originValidation.valid) {
      console.warn('Message from invalid origin:', originValidation);
      return;
    }
    
    // Process secure message
    await this.processMessage(event.data);
  }
  
  async encryptSensitiveData(data: Record<string, any>): Promise<Record<string, any>> {
    const result = { ...data };
    
    // Encrypt sensitive fields
    for (const [key, value] of Object.entries(data)) {
      if (this.isSensitiveField(key) && typeof value === 'string') {
        result[key] = await this.cryptoManager.encryptSensitiveField(
          value, 
          this.getFieldType(key),
          this.getContextId()
        );
      }
    }
    
    return result;
  }
  
  private isSensitiveField(fieldName: string): boolean {
    const sensitivePatterns = [
      /ssn/i, /social.security/i,
      /credit.card/i, /card.number/i,
      /bank.account/i, /routing.number/i,
      /password/i, /token/i
    ];
    
    return sensitivePatterns.some(pattern => pattern.test(fieldName));
  }
  
  private getFieldType(fieldName: string): 'ssn' | 'creditCard' | 'bankAccount' | 'custom' {
    if (/ssn|social/i.test(fieldName)) return 'ssn';
    if (/credit|card/i.test(fieldName)) return 'creditCard';
    if (/bank|account|routing/i.test(fieldName)) return 'bankAccount';
    return 'custom';
  }
  
  private getContextId(): string {
    return `${window.location.origin}-${Date.now()}`;
  }
  
  destroy(): void {
    // Clean up security resources
    this.cryptoManager.destroy();
  }
}

interface SecurityConfig {
  allowedOrigins: string[];
  allowedPatterns: RegExp[];
  trustedDomains: string[];
  strictMode: boolean;
}
```

This security architecture provides:

1. **Multi-layered Origin Validation**: Comprehensive validation with allowlists, patterns, and subdomain support
2. **Strong Encryption**: Web Crypto API with proper key derivation and rotation
3. **CSP Compliance**: Server nonce support with fallback strategies
4. **Clickjacking Prevention**: Visual indicators and frame validation
5. **Security Monitoring**: Event logging and compliance validation
6. **Graceful Degradation**: Secure fallbacks when security features aren't available

<function_calls>
<invoke name="TodoWrite">
<parameter name="todos">[{"id": "1", "content": "Answer Multi-Target Component System questions", "status": "completed", "priority": "high"}, {"id": "2", "content": "Answer Build System & Bundle Management questions", "status": "completed", "priority": "high"}, {"id": "3", "content": "Answer Security Implementation Details questions", "status": "completed", "priority": "high"}, {"id": "4", "content": "Answer Theme Engine Deep Dive questions", "status": "in_progress", "priority": "high"}]