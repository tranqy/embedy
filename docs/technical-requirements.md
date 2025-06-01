# Embedy: Technical Requirements Document

## Executive Summary

This document outlines the technical architecture and implementation requirements for Embedy, a comprehensive toolkit for creating embeddable applications. Based on modern tooling and best practices, this architecture enables deployment across web components, React components, and iframe sandboxes while maintaining consistent functionality and complete brandability.

### Architectural Philosophy: Producer vs Consumer Control

**Producers (Embedy App Developers)**:
- Build and compose their own screens, flows, and component hierarchies
- Define all structural elements including headers, footers, and navigation
- Implement business logic, validation rules, and data processing
- Create custom components and integrate third-party libraries
- Control the application's functional behavior and capabilities

**Consumers (Host Applications)**:
- Extensive visual customization through comprehensive theming
- Brand all UI elements via CSS custom properties and design tokens
- Configure responsive layouts, spacing, and typography
- Apply dark mode and accessibility preferences
- Cannot modify structural components or application flow

## Technology Stack

### Core Framework Selection

#### Web Components Framework: **Lit 3.x**
**Rationale**: Lit has emerged as the leading lightweight web components framework, offering superior performance and developer experience.

**Key Features**:
- TypeScript-first development with full type safety
- Reactive property system with efficient DOM updates
- Small bundle size (core: ~15KB gzipped)
- Shadow DOM isolation with configurable encapsulation
- Excellent browser compatibility (IE11+ with polyfills)

**Implementation**:
```typescript
// Base component architecture with producer/consumer separation
import { LitElement, html, css } from 'lit';
import { customElement, property } from 'lit/decorators.js';

@customElement('embedy-base')
export class EmbedyBase extends LitElement {
  // Consumer controls: theming and visual customization
  @property({ type: Object }) theme = {};
  @property({ type: String }) isolationLevel = 'shadow';
  
  // Producer controls: component structure and behavior
  protected abstract renderStructure(): TemplateResult;
  protected abstract defineBusinessLogic(): void;
  
  static styles = css`/* Dynamic theme injection from consumer */`;
}
```

#### React Components: **React 18+ with Modern Patterns**
**Rationale**: Maintain compatibility with the React ecosystem while leveraging concurrent features and modern hooks.

**Key Features**:
- React 18 concurrent rendering
- TypeScript with strict mode
- React Hook Form + Zod validation
- Storybook 8.x for documentation and testing
- Tree-shakeable component exports

#### Build System: **Vite 5.x**
**Rationale**: Vite has become the dominant build tool, offering superior development experience and production optimization.

**Key Features**:
- esbuild for blazing-fast development
- Rollup for optimized production builds
- Built-in TypeScript support
- Native ES modules in development
- Plugin ecosystem for specialized needs

**Configuration**:
```typescript
// vite.config.ts
export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, 'src/index.ts'),
      formats: ['es', 'umd', 'cjs'],
      fileName: (format) => `embedy.${format}.js`
    },
    rollupOptions: {
      external: ['react', 'react-dom'],
      output: {
        globals: {
          react: 'React',
          'react-dom': 'ReactDOM'
        }
      }
    }
  }
});
```

### Schema Validation and Forms

#### Primary Validation: **Zod 3.x**
**Rationale**: Zod has become the leading TypeScript-first validation library, offering excellent React Hook Form integration.

**Key Features**:
- Runtime type validation with TypeScript inference
- Comprehensive validation rules and custom refinements
- Excellent error messages and internationalization
- Zero dependencies and small bundle size

#### Form Management: **React Hook Form 7.x**
**Rationale**: Minimal re-renders, excellent performance, and native Zod integration.

**Implementation**:
```typescript
// Form schema definition
const invoiceSchema = z.object({
  client: z.object({
    id: z.string().uuid(),
    name: z.string().min(1, "Client name required")
  }),
  items: z.array(z.object({
    description: z.string().min(1),
    amount: z.number().positive(),
    taxRate: z.number().min(0).max(1)
  })).min(1, "At least one item required"),
  total: z.number().positive()
});

// React Hook Form integration
const { register, handleSubmit, formState: { errors } } = useForm({
  resolver: zodResolver(invoiceSchema),
  mode: 'onChange'
});
```

### Theming and Design System

#### CSS Custom Properties + Design Tokens
**Rationale**: CSS custom properties provide the most flexible and performant theming solution while maintaining clear producer/consumer boundaries.

**Architecture**:
```css
/* Design token foundation - all tokens are consumer-configurable */
:root {
  /* Consumer-controlled brand tokens */
  --embedy-color-primary: #007bff;
  --embedy-color-secondary: #6c757d;
  --embedy-color-success: #28a745;
  --embedy-color-danger: #dc3545;
  
  /* Consumer-controlled semantic tokens */
  --embedy-button-background: var(--embedy-color-primary);
  --embedy-button-color: white;
  --embedy-button-border-radius: 8px;
  
  /* Consumer-controlled spacing system */
  --embedy-space-xs: 4px;
  --embedy-space-sm: 8px;
  --embedy-space-md: 16px;
  --embedy-space-lg: 24px;
  --embedy-space-xl: 32px;
  
  /* Consumer-controlled typography scale */
  --embedy-font-family: 'Inter', system-ui, sans-serif;
  --embedy-font-size-xs: 12px;
  --embedy-font-size-sm: 14px;
  --embedy-font-size-md: 16px;
  --embedy-font-size-lg: 18px;
  --embedy-font-size-xl: 20px;
  
  /* Producer-defined structural tokens (not exposed to consumers) */
  /* These control layout and component structure, not appearance */
}

/* Consumer-controlled dark mode support */
@media (prefers-color-scheme: dark) {
  :root {
    --embedy-color-background: #1a1a1a;
    --embedy-color-surface: #2d2d2d;
    --embedy-color-text: #ffffff;
  }
}
```

#### Runtime Theme Engine
```typescript
// Theme management system with clear consumer boundaries
interface ThemeConfig {
  // All properties are consumer-controlled visual theming
  brand: {
    primaryColor: string;
    secondaryColor: string;
    fontFamily: string;
    logoUrl?: string;  // Logo image only, not structural changes
    borderRadius: string;
  };
  components: Record<string, CSSProperties>;  // Visual styles only
  layout: {
    spacing: 'compact' | 'comfortable' | 'spacious';  // Spacing, not structure
    direction: 'ltr' | 'rtl';
    maxWidth: string;
  };
  darkMode: {
    enabled: boolean;
    strategy: 'class' | 'media' | 'manual';
    colors: Record<string, string>;
  };
}

// Producer-controlled configuration (not exposed to consumers)
interface ProducerConfig {
  components: string[];  // Which components to include
  flows: FlowDefinition[];  // Application flow and logic
  structure: StructureDefinition;  // Headers, footers, navigation
  validation: ValidationRules;  // Business rules
  integrations: IntegrationConfig;  // APIs and data sources
}

class ThemeEngine {
  // Consumers can only update visual properties
  updateTheme(config: Partial<ThemeConfig>) {
    const root = document.documentElement;
    
    // Update CSS custom properties (visual only)
    Object.entries(this.flattenTheme(config)).forEach(([key, value]) => {
      root.style.setProperty(`--embedy-${key}`, value);
    });
    
    // Trigger theme change event
    window.dispatchEvent(new CustomEvent('embedy:theme-change', {
      detail: config
    }));
  }
  
  // Producer methods for structural control (not exposed)
  private defineStructure(structure: StructureDefinition) { /* ... */ }
  private implementFlow(flow: FlowDefinition) { /* ... */ }
}
```

### Navigation Isolation and Consistency

#### Navigation Architecture
**Core Principle**: Embedy applications maintain independent navigation that is visually distinct and functionally isolated from the host application.

```typescript
// Navigation isolation strategy
interface NavigationConfig {
  // Producer-controlled navigation structure
  type: 'menu' | 'tabs' | 'breadcrumb' | 'stepper';
  position: 'top' | 'left' | 'bottom' | 'embedded';
  behavior: 'push' | 'replace' | 'overlay';
  
  // Isolation mechanisms
  isolation: {
    visualBoundary: boolean;  // Clear visual separation from host
    stateManagement: 'internal' | 'url-hash' | 'postMessage';
    conflictResolution: 'namespace' | 'shadow-dom' | 'iframe';
  };
  
  // Consumer theming (visual only, not structural)
  theme: {
    backgroundColor: string;
    borderStyle: string;
    spacing: string;
    typography: TypographyConfig;
  };
}

class NavigationManager {
  private namespace = 'embedy';
  private currentState: NavigationState;
  
  // Prevent conflicts with host navigation
  initializeNavigation(config: NavigationConfig) {
    // Namespace all navigation events
    this.setupEventListeners(`${this.namespace}:navigate`);
    
    // Create visual boundary
    if (config.isolation.visualBoundary) {
      this.createNavigationContainer(config);
    }
    
    // Initialize state management
    switch (config.isolation.stateManagement) {
      case 'internal':
        this.useInternalState();
        break;
      case 'url-hash':
        this.useHashRouter(`#${this.namespace}/`);
        break;
      case 'postMessage':
        this.usePostMessageRouter();
        break;
    }
  }
  
  // Handle navigation without affecting host
  navigate(route: string, options?: NavigationOptions) {
    // Always scope navigation to embedy context
    const scopedRoute = `${this.namespace}/${route}`;
    
    // Emit scoped navigation event
    this.emit('embedy:before-navigate', { route: scopedRoute });
    
    // Update only embedy's navigation state
    this.updateNavigationState(scopedRoute);
    
    // Visual feedback within embedy boundary
    this.updateActiveIndicators(scopedRoute);
  }
}
```

#### Visual Boundary Requirements

```css
/* Clear visual separation for embedded navigation */
.embedy-navigation {
  /* Producer-defined structure */
  position: relative;
  z-index: var(--embedy-navigation-z-index, 100);
  
  /* Visual boundary (consumer can theme but not remove) */
  border: 1px solid var(--embedy-color-navigation-border, #e0e0e0);
  background: var(--embedy-color-navigation-bg, #ffffff);
  box-shadow: var(--embedy-navigation-shadow, 0 2px 4px rgba(0,0,0,0.1));
  
  /* Ensure containment */
  contain: layout style paint;
  isolation: isolate;
}

/* Prevent style leakage */
.embedy-navigation * {
  /* Reset inherited styles from host */
  all: unset; /* Changed from 'revert' for better browser support */
  font-family: var(--embedy-font-family);
}
```

#### Navigation Patterns

```typescript
// Different navigation patterns for different use cases
export const navigationPatterns = {
  // Embedded menu button with dropdown
  embeddedMenu: {
    trigger: 'menu-button',
    overlay: 'dropdown',
    position: 'relative',
    conflicts: 'low'  // Minimal conflict with host
  },
  
  // Tab-based navigation
  tabNavigation: {
    display: 'horizontal-tabs',
    position: 'top',
    conflicts: 'medium'  // May compete visually with host tabs
  },
  
  // Sidebar navigation
  sidebarNavigation: {
    display: 'vertical-menu',
    position: 'left',
    overlay: 'push-content',
    conflicts: 'high'  // Requires dedicated space
  },
  
  // Stepper/wizard navigation
  stepperNavigation: {
    display: 'progress-stepper',
    position: 'top',
    linear: true,
    conflicts: 'low'  // Self-contained, minimal conflict
  }
};
```

### Security and Isolation

#### Iframe Sandboxing
**Implementation based on modern security best practices**:

```typescript
// Security adapter based on environment detection
class SecurityAdapter {
  private readonly sandboxConfig = {
    IFRAME_SANDBOX: [
      'allow-same-origin',
      'allow-scripts',
      'allow-forms',
      'allow-popups'
    ],
    SHADOW_DOM_ISOLATED: {
      mode: 'closed',
      delegatesFocus: true
    },
    COMPONENT_BOUNDARY: {
      isolation: 'style-only'
    }
  };

  determineSandboxLevel(environment: HostEnvironment): IsolationLevel {
    // Enhanced detection based on modern security requirements
    if (environment.hasStrictCSP || environment.handlesPayments) {
      return 'IFRAME_SANDBOX';
    }
    
    if (environment.hasDataPrivacyReqs || environment.isThirdParty) {
      return 'SHADOW_DOM_ISOLATED';
    }
    
    return 'COMPONENT_BOUNDARY';
  }
}
```

#### Comprehensive Security Implementation

**Security Threat Model**: Embedy components must protect against XSS, clickjacking, data exfiltration, and privacy violations while maintaining functionality across isolation levels.

##### Content Security Policy (CSP) Compliance
```html
<!-- Minimum required CSP directives for iframe embedding -->
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self';
  script-src 'self' 'unsafe-inline' https://cdn.embedy.com;
  style-src 'self' 'unsafe-inline';
  frame-src 'self' https://embed.embedy.com;
  frame-ancestors 'self' https://trusted-host.com;
  connect-src 'self' https://api.embedy.com;
">

<!-- Strict CSP for high-security environments -->
<meta http-equiv="Content-Security-Policy" content="
  default-src 'none';
  script-src 'self' 'nonce-{random}';
  style-src 'self' 'nonce-{random}';
  frame-src 'self';
  frame-ancestors 'none';
">
```

##### Clickjacking Protection
```typescript
// Iframe embedding protection
class ClickjackingProtection {
  static validateEmbeddingContext() {
    // Prevent embedding in untrusted contexts
    if (window.top !== window.self) {
      const parentOrigin = document.referrer;
      const allowedOrigins = ['https://trusted-host.com', 'https://partner.com'];
      
      if (!allowedOrigins.some(origin => parentOrigin.startsWith(origin))) {
        throw new Error('Embedding not allowed from this origin');
      }
    }
  }
  
  static setFrameOptions() {
    // X-Frame-Options header equivalent
    if (window.top === window.self) {
      document.head.appendChild(Object.assign(document.createElement('meta'), {
        httpEquiv: 'X-Frame-Options',
        content: 'SAMEORIGIN'
      }));
    }
  }
}
```

##### Data Privacy and GDPR Compliance
```typescript
interface PrivacyConfig {
  dataProcessingBasis: 'consent' | 'legitimate_interest' | 'contract';
  dataRetentionDays: number;
  anonymizeData: boolean;
  crossBorderTransfer: boolean;
  encryptionRequired: boolean;
}

class PrivacyManager {
  private config: PrivacyConfig;
  
  validateDataHandling(formData: FormData) {
    // Encrypt sensitive fields before transmission
    const sensitiveFields = ['ssn', 'creditCard', 'bankAccount'];
    
    Object.keys(formData).forEach(key => {
      if (sensitiveFields.includes(key)) {
        if (!this.config.encryptionRequired) {
          throw new Error(`Field ${key} requires encryption`);
        }
        formData[key] = this.encryptField(formData[key]);
      }
    });
    
    // Add privacy metadata
    return {
      ...formData,
      _privacy: {
        consentTimestamp: Date.now(),
        processingBasis: this.config.dataProcessingBasis,
        retentionExpiry: Date.now() + (this.config.dataRetentionDays * 86400000)
      }
    };
  }
  
  private encryptField(value: string): string {
    // Use Web Crypto API for client-side encryption
    return crypto.subtle.encrypt('AES-GCM', this.getEncryptionKey(), 
      new TextEncoder().encode(value));
  }
}
```

#### PostMessage Security

**Library Recommendation**: **Postmate** by Dollar Shave Club
**Rationale**: Promise-based API with built-in security validation, minimal bundle size (~1.6KB gzipped), and excellent documentation.

**Enhanced Security Implementation**:

```typescript
// Secure PostMessage implementation with validation
class SecurePostMessage {
  private allowedOrigins: Set<string>;
  private messageValidator: MessageValidator;
  
  constructor(allowedOrigins: string[]) {
    this.allowedOrigins = new Set(allowedOrigins);
    this.messageValidator = new MessageValidator();
  }
  
  async createSecureChild(config: IframeConfig) {
    // Enhanced Postmate configuration with security validation
    const handshake = new Postmate({
      container: config.container,
      url: config.url,
      classListArray: ['embedy-secure-iframe'],
      
      // Security enhancements
      model: {
        // Validate all incoming messages
        validateMessage: (data: any) => {
          return this.messageValidator.validate(data);
        },
        
        // Secure data transmission
        sendSecureData: (data: any) => {
          const encrypted = this.encryptMessage(data);
          return this.sendWithIntegrity(encrypted);
        }
      }
    });
    
    return handshake.then(child => {
      // Additional security setup
      this.setupSecurityHeaders(child);
      this.enableIntegrityChecking(child);
      return child;
    });
  }
  
  private validateOrigin(origin: string): boolean {
    return this.allowedOrigins.has(origin) || 
           this.allowedOrigins.has('*'); // Only for development
  }
  
  private encryptMessage(data: any): EncryptedMessage {
    // Use Web Crypto API for message encryption
    const key = crypto.getRandomValues(new Uint8Array(32));
    return {
      payload: crypto.subtle.encrypt('AES-GCM', key, JSON.stringify(data)),
      timestamp: Date.now(),
      nonce: crypto.getRandomValues(new Uint8Array(12))
    };
  }
}

class MessageValidator {
  private schema: MessageSchema;
  
  validate(message: any): ValidationResult {
    // Validate message structure and content
    const schemaValidation = this.schema.safeParse(message);
    
    if (!schemaValidation.success) {
      throw new SecurityError('Invalid message format', {
        errors: schemaValidation.error.errors,
        receivedData: message
      });
    }
    
    // Additional security checks
    this.validateMessageSize(message);
    this.validateMessageFrequency(message);
    this.scanForMaliciousContent(message);
    
    return { valid: true, sanitizedData: schemaValidation.data };
  }
  
  private scanForMaliciousContent(message: any): void {
    // Basic XSS prevention
    const dangerousPatterns = [
      /<script[^>]*>.*?<\/script>/gi,
      /javascript:/gi,
      /on\w+\s*=/gi,
      /data:text\/html/gi
    ];
    
    const messageStr = JSON.stringify(message);
    dangerousPatterns.forEach(pattern => {
      if (pattern.test(messageStr)) {
        throw new SecurityError('Potentially malicious content detected');
      }
    });
  }
}
```

##### Cross-Origin Resource Sharing (CORS) Configuration
```typescript
// CORS policy for API endpoints
const corsConfig = {
  // Production configuration
  production: {
    origin: ['https://trusted-partner.com', 'https://client-domain.com'],
    methods: ['GET', 'POST'],
    allowedHeaders: ['Content-Type', 'Authorization', 'X-Embedy-Token'],
    credentials: true,
    maxAge: 86400 // 24 hours
  },
  
  // Development configuration  
  development: {
    origin: true, // Allow all origins in development
    methods: ['GET', 'POST', 'OPTIONS'],
    allowedHeaders: ['*'],
    credentials: true
  }
};

class CORSValidator {
  static validateRequest(request: Request, config: CORSConfig): boolean {
    const origin = request.headers.get('origin');
    
    if (!origin) {
      throw new SecurityError('Missing origin header');
    }
    
    if (Array.isArray(config.origin)) {
      return config.origin.includes(origin);
    }
    
    return config.origin === true || config.origin === origin;
  }
}
```

##### Security Monitoring and Alerting
```typescript
class SecurityMonitor {
  private metrics: SecurityMetrics;
  private alertThresholds: AlertThresholds;
  
  logSecurityEvent(event: SecurityEvent) {
    // Log security events for monitoring
    console.warn('[EMBEDY SECURITY]', {
      type: event.type,
      severity: event.severity,
      origin: event.origin,
      timestamp: Date.now(),
      userAgent: navigator.userAgent,
      details: event.details
    });
    
    // Send to monitoring service
    this.sendToMonitoring(event);
    
    // Check if immediate action required
    if (event.severity === 'high') {
      this.triggerSecurityAlert(event);
    }
  }
  
  private triggerSecurityAlert(event: SecurityEvent) {
    // Disable further operations if severe security threat
    if (event.type === 'MALICIOUS_CONTENT' || event.type === 'ORIGIN_VIOLATION') {
      this.disableEmbeddedComponent();
      this.notifyParentWindow('SECURITY_ALERT', event);
    }
  }
}
```

Using Postmate for secure iframe communication

// Parent implementation
const handshake = new Postmate({
  container: document.getElementById('iframe-container'),
  url: 'https://child-origin.com/iframe.html',
  classListArray: ['custom-iframe-class']
});

handshake.then(child => {
  // Listen to events from child
  child.on('form-submitted', data => {
    console.log('Received form data:', data);
  });
  
  // Send data to child
  child.call('updateTheme', { primaryColor: '#007bff' });
});

// Child iframe implementation  
const handshake = new Postmate.Model({
  // Expose methods to parent
  updateTheme: (themeConfig) => {
    applyTheme(themeConfig);
    return 'Theme updated successfully';
  },
  
  // Expose data to parent
  getFormData: () => {
    return currentFormState;
  }
});

// Emit events to parent
handshake.then(parent => {
  parent.emit('form-submitted', formData);
});
```

**Alternative Libraries**:
- **PostMessenger**: Simple wrapper with domain verification (~1.5KB)
- **please.js**: Request/Response pattern with jQuery integration
- **Interframe**: Lightweight with message queuing support

**Security Features**:
- Origin validation and domain restrictions
- Message structure validation
- Promise-based request/response pattern
- Automatic handshake verification

### Performance Optimization

#### Bundle Splitting Strategy
```typescript
// Dynamic imports for progressive enhancement
class FeatureLoader {
  private capabilities: DeviceCapabilities;

  async loadOptimalFeatureSet(): Promise<ComponentLibrary> {
    // Core features (always loaded)
    const core = await import('./core/forms');
    
    // Progressive enhancement based on capabilities
    const features = await Promise.all([
      this.capabilities.hasModernBrowser && this.capabilities.bandwidth > '3G' 
        ? import('./features/advanced-validation')
        : Promise.resolve(null),
      
      this.capabilities.touchDevice 
        ? import('./features/mobile-optimizations')
        : Promise.resolve(null),
        
      this.capabilities.supportsIntersectionObserver
        ? import('./features/virtual-scrolling')
        : Promise.resolve(null)
    ]);

    return this.assembleLibrary(core, features.filter(Boolean));
  }
}
```

#### Performance Targets
- **Core Framework**: 18KB gzipped (includes polyfills for IE11+)
- **Individual Components**: 3-8KB gzipped each
- **Complete Form Suite**: 45KB gzipped
- **Time to Interactive**: < 3 seconds on 3G
- **Component Render**: < 100ms for complex forms (realistic with validation)
- **Memory Usage**: < 15MB for complete form suite in production
- **Bundle Analysis**: Automatic size regression detection with 10% tolerance

### Documentation and Developer Experience

#### Storybook 8.x Configuration
```typescript
// .storybook/main.ts
export default {
  framework: '@storybook/web-components-vite',
  stories: ['../src/**/*.stories.@(js|jsx|ts|tsx|mdx)'],
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-a11y',
    '@storybook/addon-design-tokens'
  ],
  docs: {
    autodocs: 'tag',
    defaultName: 'Documentation'
  }
};
```

#### Type Definitions
```typescript
// Complete TypeScript definitions with producer/consumer separation
export interface EmbedyComponent {
  // Consumer-controlled properties
  theme: ThemeConfig;  // Visual theming only
  isolationLevel: IsolationLevel;
  
  // Producer-defined callbacks (structure maintained by producer)
  onSubmit?: (data: FormData, validation: ValidationResult) => void;
  onChange?: (fieldId: string, value: unknown) => void;
  onError?: (error: FormError, context: ErrorContext) => void;
}

export interface FormConfig {
  // Producer-controlled structure
  id: string;
  title: string;
  fields: FieldDefinition[];  // Producer defines fields and logic
  layout: LayoutConfig;  // Producer defines structure, consumer themes it
  validation: ValidationConfig;  // Producer business rules
  
  // Consumer-controlled branding
  branding: BrandingConfig;  // Visual customization only
}

// Clear separation in component definition
export interface ComponentDefinition {
  // Producer domain: what the component does
  producerAspects: {
    structure: ComponentStructure;  // DOM hierarchy
    behavior: ComponentBehavior;  // Event handlers, logic
    dataFlow: DataFlowDefinition;  // State management
    validation: ValidationRules;  // Business rules
  };
  
  // Consumer domain: how the component looks
  consumerAspects: {
    theme: ThemeConfig;  // Colors, typography, spacing
    icons: IconSet;  // Visual icons
    labels: LabelConfig;  // Text content (i18n)
    animations: AnimationConfig;  // Visual transitions
  };
}
```

## Complete API Reference

### Core Component API

#### EmbedyBase Component
```typescript
@customElement('embedy-base')
export class EmbedyBase extends LitElement {
  // Configuration properties
  @property({ type: Object }) theme: ThemeConfig = {};
  @property({ type: String }) isolationLevel: IsolationLevel = 'shadow';
  @property({ type: String }) apiEndpoint: string = '';
  @property({ type: String }) authToken: string = '';
  @property({ type: Boolean }) disabled: boolean = false;
  @property({ type: Object }) validationRules: ValidationConfig = {};

  // Event callbacks with error handling
  @property({ type: Function }) 
  onSubmit?: (data: FormData, context: SubmissionContext) => Promise<SubmissionResult>;
  
  @property({ type: Function }) 
  onChange?: (fieldId: string, value: unknown, validation: ValidationResult) => void;
  
  @property({ type: Function }) 
  onError?: (error: EmbedyError, context: ErrorContext) => void;
  
  @property({ type: Function }) 
  onLoad?: (component: EmbedyBase, loadTime: number) => void;

  // Lifecycle methods with error boundaries
  async connectedCallback() {
    super.connectedCallback();
    
    try {
      await this.initializeComponent();
      this.dispatchEvent(new CustomEvent('embedy:ready', {
        detail: { component: this, timestamp: Date.now() }
      }));
    } catch (error) {
      this.handleError(error, 'INITIALIZATION_ERROR');
    }
  }

  async disconnectedCallback() {
    try {
      await this.cleanup();
    } catch (error) {
      console.warn('Cleanup error:', error);
    }
    super.disconnectedCallback();
  }

  // Public API methods
  async updateTheme(themeConfig: Partial<ThemeConfig>): Promise<void> {
    try {
      this.validateThemeConfig(themeConfig);
      this.theme = { ...this.theme, ...themeConfig };
      await this.applyTheme();
    } catch (error) {
      throw new EmbedyError('THEME_UPDATE_FAILED', {
        originalError: error,
        themeConfig,
        currentTheme: this.theme
      });
    }
  }

  async validate(): Promise<ValidationResult> {
    try {
      const result = await this.performValidation();
      this.dispatchEvent(new CustomEvent('embedy:validation', {
        detail: result
      }));
      return result;
    } catch (error) {
      const validationError = new EmbedyError('VALIDATION_FAILED', {
        originalError: error,
        formData: this.getFormData()
      });
      this.handleError(validationError, 'VALIDATION_ERROR');
      throw validationError;
    }
  }

  async submit(): Promise<SubmissionResult> {
    try {
      // Pre-submission validation
      const validationResult = await this.validate();
      if (!validationResult.valid) {
        throw new EmbedyError('VALIDATION_FAILED', {
          errors: validationResult.errors
        });
      }

      // Submit data
      const formData = this.getFormData();
      const result = await this.submitData(formData);
      
      this.dispatchEvent(new CustomEvent('embedy:submit-success', {
        detail: { result, formData }
      }));
      
      return result;
    } catch (error) {
      const submissionError = new EmbedyError('SUBMISSION_FAILED', {
        originalError: error,
        formData: this.getFormData(),
        validationState: await this.validate().catch(() => null)
      });
      
      this.handleError(submissionError, 'SUBMISSION_ERROR');
      throw submissionError;
    }
  }

  // Error handling
  private handleError(error: EmbedyError, context: ErrorContext): void {
    // Log error for debugging
    console.error('[EMBEDY ERROR]', {
      type: error.type,
      message: error.message,
      context,
      component: this.tagName,
      timestamp: Date.now()
    });

    // Call user-provided error handler
    if (this.onError) {
      try {
        this.onError(error, context);
      } catch (handlerError) {
        console.error('Error in user error handler:', handlerError);
      }
    }

    // Dispatch error event
    this.dispatchEvent(new CustomEvent('embedy:error', {
      detail: { error, context },
      bubbles: true
    }));

    // Update UI to show error state
    this.showErrorState(error);
  }
}
```

#### Error Types and Handling
```typescript
// Comprehensive error system
export class EmbedyError extends Error {
  constructor(
    public type: ErrorType,
    public details: ErrorDetails = {},
    message?: string
  ) {
    super(message || EmbedyError.getDefaultMessage(type));
    this.name = 'EmbedyError';
  }

  static getDefaultMessage(type: ErrorType): string {
    const messages: Record<ErrorType, string> = {
      'INITIALIZATION_ERROR': 'Failed to initialize component',
      'VALIDATION_FAILED': 'Form validation failed',
      'SUBMISSION_FAILED': 'Form submission failed',
      'THEME_UPDATE_FAILED': 'Failed to update theme',
      'NETWORK_ERROR': 'Network request failed',
      'SECURITY_VIOLATION': 'Security policy violation',
      'CONFIGURATION_ERROR': 'Invalid configuration provided'
    };
    return messages[type] || 'Unknown error occurred';
  }

  toJSON() {
    return {
      type: this.type,
      message: this.message,
      details: this.details,
      stack: this.stack,
      timestamp: Date.now()
    };
  }
}

// Error boundaries for React components
export class EmbedyErrorBoundary extends React.Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return {
      hasError: true,
      error: error instanceof EmbedyError ? error : new EmbedyError(
        'COMPONENT_ERROR',
        { originalError: error }
      )
    };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log error to monitoring service
    this.logErrorToService(error, errorInfo);
    
    // Call user-provided error handler
    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="embedy-error-state">
          <h3>Something went wrong</h3>
          <p>Please try refreshing the page or contact support.</p>
          <details>
            <summary>Error details</summary>
            <pre>{JSON.stringify(this.state.error?.toJSON(), null, 2)}</pre>
          </details>
        </div>
      );
    }

    return this.props.children;
  }
}
```

#### Network and API Layer
```typescript
// Robust API client with retry logic and error handling
export class EmbedyApiClient {
  private baseUrl: string;
  private authToken: string;
  private retryConfig: RetryConfig;

  constructor(config: ApiClientConfig) {
    this.baseUrl = config.baseUrl;
    this.authToken = config.authToken;
    this.retryConfig = config.retryConfig || {
      maxRetries: 3,
      backoffMs: 1000,
      retryableStatuses: [408, 429, 500, 502, 503, 504]
    };
  }

  async submitForm(formData: FormData): Promise<SubmissionResult> {
    return this.retryRequest(async () => {
      const response = await fetch(`${this.baseUrl}/submit`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${this.authToken}`,
          'X-Embedy-Version': '1.0'
        },
        body: JSON.stringify(formData)
      });

      if (!response.ok) {
        throw new EmbedyError('SUBMISSION_FAILED', {
          status: response.status,
          statusText: response.statusText,
          url: response.url
        });
      }

      return response.json();
    });
  }

  private async retryRequest<T>(
    requestFn: () => Promise<T>
  ): Promise<T> {
    let lastError: Error;
    
    for (let attempt = 0; attempt <= this.retryConfig.maxRetries; attempt++) {
      try {
        return await requestFn();
      } catch (error) {
        lastError = error;
        
        // Don't retry on client errors (4xx except specific ones)
        if (error instanceof EmbedyError && 
            error.details.status >= 400 && 
            error.details.status < 500 &&
            !this.retryConfig.retryableStatuses.includes(error.details.status)) {
          throw error;
        }

        // Wait before retry (exponential backoff)
        if (attempt < this.retryConfig.maxRetries) {
          const delay = this.retryConfig.backoffMs * Math.pow(2, attempt);
          await new Promise(resolve => setTimeout(resolve, delay));
        }
      }
    }

    throw new EmbedyError('NETWORK_ERROR', {
      originalError: lastError,
      attempts: this.retryConfig.maxRetries + 1
    });
  }
}
```

### Integration Examples with Error Handling

#### React Integration
```typescript
// Complete React integration example
import { EmbedyProvider, InvoiceForm, EmbedyErrorBoundary } from '@embedy/react';

function App() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<EmbedyError | null>(null);

  const handleSubmit = async (data: FormData) => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await apiClient.submitForm(data);
      console.log('Form submitted successfully:', result);
      // Handle success (redirect, show success message, etc.)
    } catch (error) {
      console.error('Form submission failed:', error);
      setError(error as EmbedyError);
    } finally {
      setLoading(false);
    }
  };

  const handleError = (error: EmbedyError, context: ErrorContext) => {
    // Custom error handling logic
    setError(error);
    
    // Send error to monitoring service
    errorMonitoring.captureError(error, context);
  };

  return (
    <EmbedyErrorBoundary 
      onError={handleError}
      fallback={<ErrorFallback />}>
      <EmbedyProvider 
        theme={customTheme}
        apiConfig={{
          baseUrl: process.env.REACT_APP_EMBEDY_API_URL,
          authToken: process.env.REACT_APP_EMBEDY_TOKEN
        }}>
        
        <InvoiceForm
          onSubmit={handleSubmit}
          onError={handleError}
          loading={loading}
          disabled={loading}
          validationRules={{
            realTimeValidation: true,
            showErrorsOnBlur: true
          }}
        />
        
        {error && (
          <ErrorDisplay 
            error={error} 
            onRetry={() => setError(null)} 
          />
        )}
      </EmbedyProvider>
    </EmbedyErrorBoundary>
  );
}
```

#### Web Components Integration
```html
<!-- Complete web components integration -->
<script>
document.addEventListener('DOMContentLoaded', () => {
  const invoiceForm = document.querySelector('embedy-invoice-form');
  
  // Configure error handling
  invoiceForm.onError = (error, context) => {
    console.error('Embedy error:', error);
    
    // Show user-friendly error message
    showErrorToast(error.message);
    
    // Send to error tracking
    if (window.Sentry) {
      window.Sentry.captureException(error);
    }
  };
  
  // Configure submission handling
  invoiceForm.onSubmit = async (data, context) => {
    try {
      const response = await fetch('/api/invoices', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      const result = await response.json();
      showSuccessMessage('Invoice created successfully!');
      return result;
    } catch (error) {
      // Error will be handled by onError callback
      throw error;
    }
  };
  
  // Handle component events
  invoiceForm.addEventListener('embedy:validation', (event) => {
    const { valid, errors } = event.detail;
    updateValidationUI(valid, errors);
  });
  
  invoiceForm.addEventListener('embedy:error', (event) => {
    const { error, context } = event.detail;
    logErrorForDebugging(error, context);
  });
});

// Utility functions
function showErrorToast(message) {
  // Implementation depends on your toast library
  console.error('Error:', message);
}

function showSuccessMessage(message) {
  // Implementation depends on your notification system
  console.log('Success:', message);
}

function updateValidationUI(valid, errors) {
  // Update form validation state in UI
  const submitButton = document.querySelector('#submit-button');
  submitButton.disabled = !valid;
}
</script>

<embedy-invoice-form 
  api-endpoint="/api/invoices"
  theme="custom-brand"
  isolation-level="shadow">
</embedy-invoice-form>
```

## Implementation Phases

### Phase 1: Foundation (Months 1-3)
- **Week 1-2**: Project setup with Vite, TypeScript, and Lit
- **Week 3-4**: Core theming system and design tokens
- **Week 5-6**: Basic web components (input, button, form container)
- **Week 7-8**: React wrapper components with Hook Form integration
- **Week 9-10**: Iframe embedding and security framework
- **Week 11-12**: Testing infrastructure and initial Storybook setup

### Phase 2: Component Library (Months 4-6)
- **Month 4**: Complete form components (validation, formatting, business logic)
- **Month 5**: Advanced theming system and runtime customization
- **Month 6**: Performance optimization and bundle analysis

### Phase 3: Polish and Ecosystem (Months 7-9)
- **Month 7**: Developer documentation and integration guides
- **Month 8**: Advanced security features and compliance tools
- **Month 9**: Community feedback integration and v1.0 release

## Success Metrics

### Technical Metrics
- **Bundle Size**: Core < 50KB, complete suite < 150KB gzipped
- **Performance**: Lighthouse score > 95
- **Compatibility**: IE11+ (with polyfills), all modern browsers
- **Type Safety**: 100% TypeScript coverage
- **Test Coverage**: > 90% unit test coverage

### Developer Experience Metrics
- **Integration Time**: < 30 minutes from install to working component
- **Documentation Coverage**: Every component and API documented
- **Community Engagement**: GitHub stars, NPM downloads, community contributions

## Risk Mitigation

### Technical Risks
1. **Browser Compatibility**: Comprehensive polyfill strategy and feature detection
2. **Bundle Size Creep**: Automated bundle analysis in CI/CD
3. **Security Vulnerabilities**: Regular security audits and dependency updates
4. **Performance Regression**: Continuous performance monitoring

### Adoption Risks
1. **Learning Curve**: Comprehensive documentation and examples
2. **Migration Path**: Clear upgrade guides and compatibility layers
3. **Ecosystem Integration**: Support for popular frameworks and tools

## Conclusion

This technical architecture leverages modern and battle-tested tools to create a robust, performant, and developer-friendly embeddable component system. The combination of Lit for web components, React Hook Form + Zod for validation, Vite for building, Postmate for secure iframe communication, and CSS custom properties for theming provides a solid foundation that can scale from simple integrations to complex enterprise deployments.