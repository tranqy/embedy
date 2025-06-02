# Embedy Code Examples

This document contains all code examples for the Embedy toolkit, organized by feature and integration method.

## Basic Integration Examples

### Web Components (Native)

#### Simple Embedding
```javascript
// Direct DOM integration with shadow DOM isolation
<embedy-invoice-form 
  theme="custom-brand"
  api-endpoint="/api/invoices"
  isolation-level="shadow">
</embedy-invoice-form>
```

### React Components

#### Framework Integration
```javascript
// Framework-native integration with auto-generated React wrapper
import { InvoiceForm } from '@embedy/react';

<InvoiceForm 
  theme={customTheme}
  onSubmit={handleSubmit}
  isolationLevel="component"
  apiEndpoint="/api/invoices"
  validationRules={{
    realTimeValidation: true,
    showErrorsOnBlur: true
  }}
/>
```

#### Multi-Target Component Definition
```typescript
// Web component that automatically generates React wrapper
@defineEmbedyComponent<InvoiceFormProps>('embedy-invoice-form', InvoiceFormComponent)
export class InvoiceFormComponent extends EmbedyBase implements EmbedyComponentInterface {
  @embedyProperty({ type: String })
  apiEndpoint: string = '';
  
  @embedyProperty({ type: Object })
  validationRules: ValidationConfig = {};
  
  @embedyProperty({ type: Function })
  onSubmit?: (data: FormData) => Promise<SubmissionResult>;
  
  async validate(): Promise<ValidationResult> {
    const result = await this.validator.validate(this.getFormData());
    this.dispatchEvent(new CustomEvent('embedy:validation', { detail: result }));
    return result;
  }
  
  render() {
    return html`<div class="invoice-form"><!-- Form content --></div>`;
  }
}

// Auto-generated React wrapper (created at build time)
export const InvoiceForm = forwardRef<HTMLElement, InvoiceFormProps>((props, ref) => {
  const webComponentRef = useRef<InvoiceFormComponent>(null);
  
  // Sync props to web component
  useEffect(() => {
    const element = webComponentRef.current;
    if (element) {
      element.theme = props.theme || {};
      element.apiEndpoint = props.apiEndpoint || '';
      element.validationRules = props.validationRules || {};
    }
  }, [props.theme, props.apiEndpoint, props.validationRules]);
  
  // Handle React event adaptation
  ReactAdaptationLayer.adaptCustomEvents(webComponentRef, {
    'embedy:submit': props.onSubmit,
    'embedy:validation': props.onValidation
  });
  
  return <embedy-invoice-form ref={mergeRefs([ref, webComponentRef])} />;
});
```

### Iframe Sandbox

#### Maximum Security Isolation
```javascript
// Maximum security isolation
<embedy-iframe 
  src="/embed/invoice-form"
  sandbox="allow-same-origin allow-scripts"
  theme-url="/themes/custom.css">
</embedy-iframe>
```

## Environment Detection and Adaptation

### Framework Detection
```javascript
const EnvironmentDetector = {
  detectFramework: () => {
    return {
      react: window.React ? window.React.version : null,
      angular: window.ng ? 'detected' : null,
      vue: window.Vue ? window.Vue.version : null,
      legacy: !window.customElements
    };
  },
  
  detectConstraints: () => {
    return {
      cspRestrictions: !this.canUseInlineStyles(),
      performanceBudget: this.calculateAvailableBudget(),
      mobileEnvironment: this.isMobileWebView(),
      accessibilityRequirements: this.detectA11yNeeds()
    };
  },
  
  detectNavigationConflicts: () => {
    return {
      hasHostMenuButton: this.detectHostNavigation(),
      hasHostRouter: window.history && window.location.hash,
      hasHostModal: this.detectModalManagement(),
      zIndexRange: this.detectAvailableZIndex()
    };
  }
};
```

### Adaptive Security Model
```javascript
class SecurityAdapter {
  constructor(hostEnvironment) {
    this.isolationLevel = this.determineIsolationLevel(hostEnvironment);
    this.setupSecurityBoundaries();
  }
  
  determineIsolationLevel(env) {
    if (env.hasStrictCSP || env.handlesPayments) {
      return 'IFRAME_SANDBOX'; // Maximum isolation
    } else if (env.hasDataPrivacyReqs) {
      return 'SHADOW_DOM_ISOLATED'; // Style and DOM isolation
    } else {
      return 'COMPONENT_BOUNDARY'; // Standard component model
    }
  }
}
```

### Progressive Feature Loading
```javascript
// Enhanced capability detection and progressive loading
class ProgressiveFeatureLoader {
  constructor() {
    this.capabilities = this.detectCapabilities();
  }
  
  detectCapabilities() {
    return {
      bandwidth: this.detectBandwidth(), // 2G/3G/4G/wifi
      memory: (navigator.deviceMemory || 4), // GB
      touchDevice: 'ontouchstart' in window,
      hasModernBrowser: this.detectModernBrowser(),
      supportsWebComponents: 'customElements' in window
    };
  }
  
  async loadOptimalFeatureSet() {
    // Core features (always loaded - 18KB gzipped)
    const core = await import('./core/forms');
    
    // Progressive enhancement based on capabilities
    const features = await Promise.allSettled([
      this.loadAdvancedValidation(),
      this.loadMobileOptimizations(),
      this.loadVirtualScrolling(),
      this.loadRichFormatting()
    ]);
    
    return this.assembleLibrary(core, features.filter(r => r.status === 'fulfilled'));
  }
  
  async loadAdvancedValidation() {
    if (this.capabilities.hasModernBrowser && 
        this.capabilities.bandwidth !== '2G' &&
        this.capabilities.memory >= 2) {
      return import('./features/advanced-validation'); // +8KB
    }
    return null;
  }
  
  async loadMobileOptimizations() {
    if (this.capabilities.touchDevice) {
      return import('./features/mobile-optimizations'); // +5KB
    }
    return null;
  }
}

// Bundle splitting with fallback loading
class ChunkManager {
  async loadChunk(chunkName) {
    const fallbackPaths = [
      `/chunks/${chunkName}.js`,
      `/fallbacks/${chunkName}.js`,
      `/core/minimal-${chunkName}.js`
    ];
    
    for (const path of fallbackPaths) {
      try {
        return await import(path);
      } catch (error) {
        console.warn(`Failed to load chunk from ${path}, trying fallback`);
      }
    }
    
    throw new Error(`Failed to load chunk: ${chunkName}`);
  }
}

// Polyfill management
class PolyfillLoader {
  static async loadRequiredPolyfills(features = ['custom-elements', 'shadow-dom', 'fetch']) {
    const missingFeatures = features.filter(feature => !this.isFeatureSupported(feature));
    
    if (missingFeatures.length > 0) {
      console.info(`Loading polyfills for: ${missingFeatures.join(', ')}`);
      await Promise.allSettled(missingFeatures.map(this.loadPolyfill));
    }
  }
  
  static isFeatureSupported(feature) {
    const tests = {
      'custom-elements': () => 'customElements' in window,
      'shadow-dom': () => 'attachShadow' in Element.prototype,
      'fetch': () => 'fetch' in window,
      'intersection-observer': () => 'IntersectionObserver' in window
    };
    return tests[feature]?.() || false;
  }
  
  static async loadPolyfill(feature) {
    const polyfills = {
      'custom-elements': () => import('@webcomponents/webcomponentsjs'),
      'shadow-dom': () => import('@webcomponents/webcomponentsjs/webcomponents-bundle.js'),
      'fetch': () => import('whatwg-fetch'),
      'intersection-observer': () => import('intersection-observer')
    };
    
    try {
      await polyfills[feature]?.();
    } catch (error) {
      console.error(`Failed to load polyfill for ${feature}:`, error);
    }
  }
}
```

## Navigation Isolation Examples

### Navigation Configuration
```javascript
// Navigation configuration with isolation strategies
const navigationConfig = {
  // Embedded menu with dropdown
  embeddedMenu: {
    type: 'menu',
    position: 'embedded',
    trigger: 'menu-button',
    isolation: {
      visualBoundary: true,
      stateManagement: 'internal',
      conflictResolution: 'namespace'
    },
    theme: {
      backgroundColor: 'var(--embedy-color-navigation-bg)',
      borderColor: 'var(--embedy-color-navigation-border)',
      shadow: 'var(--embedy-navigation-shadow)'
    }
  },
  
  // Tab navigation for multi-step forms
  tabNavigation: {
    type: 'tabs',
    position: 'top',
    behavior: 'replace',
    isolation: {
      visualBoundary: true,
      stateManagement: 'url-hash',
      conflictResolution: 'shadow-dom'
    }
  },
  
  // Stepper for wizard flows
  stepperNavigation: {
    type: 'stepper',
    position: 'top',
    linear: true,
    isolation: {
      visualBoundary: false, // Self-contained visual
      stateManagement: 'internal',
      conflictResolution: 'namespace'
    }
  }
};
```

### Visual Boundary CSS
```css
/* Clear visual separation for embedded navigation */
.embedy-navigation {
  /* Visual containment */
  border: 1px solid var(--embedy-color-navigation-border, #e0e0e0);
  background: var(--embedy-color-navigation-bg, #ffffff);
  box-shadow: var(--embedy-navigation-shadow, 0 2px 4px rgba(0,0,0,0.1));
  
  /* Ensure isolation from host styles */
  contain: layout style paint;
  isolation: isolate;
  z-index: var(--embedy-navigation-z-index, 100);
}

/* Mobile-responsive navigation */
@media (max-width: 768px) {
  .embedy-navigation {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    border-top: 1px solid var(--embedy-color-navigation-border);
    border-left: none;
    border-right: none;
  }
}
```

### Navigation State Management
```javascript
// Isolated navigation manager
class EmbedyNavigationManager {
  constructor(config) {
    this.namespace = 'embedy';
    this.config = config;
    this.initializeNavigation();
  }
  
  initializeNavigation() {
    // Create scoped navigation context
    this.navigationContext = {
      routes: this.prefixRoutes(this.config.routes),
      currentPath: this.getCurrentPath(),
      history: []
    };
    
    // Setup isolated event handling
    this.setupNavigationListeners();
    
    // Initialize visual boundaries
    if (this.config.isolation.visualBoundary) {
      this.createNavigationContainer();
    }
  }
  
  navigate(route, options = {}) {
    // Always scope to embedy namespace
    const scopedRoute = `${this.namespace}/${route}`;
    
    // Handle navigation based on strategy
    switch (this.config.isolation.stateManagement) {
      case 'internal':
        this.updateInternalState(scopedRoute);
        break;
      case 'url-hash':
        window.location.hash = `#${scopedRoute}`;
        break;
      case 'postMessage':
        this.sendNavigationMessage(scopedRoute);
        break;
    }
    
    // Update visual indicators
    this.updateActiveNavItem(scopedRoute);
  }
  
  // Prevent navigation conflicts
  setupNavigationListeners() {
    // Use namespaced events
    document.addEventListener(`${this.namespace}:navigate`, (e) => {
      e.stopPropagation(); // Prevent bubbling to host
      this.navigate(e.detail.route, e.detail.options);
    });
    
    // Handle back/forward for hash routing
    if (this.config.isolation.stateManagement === 'url-hash') {
      window.addEventListener('hashchange', (e) => {
        if (e.newURL.includes(`#${this.namespace}/`)) {
          this.handleHashChange(e);
        }
      });
    }
  }
}
```

## Theming System Examples

### Multi-Level Theme Configuration
```javascript
// Theme configuration with complete brand control
const themeConfig = {
  // Brand identity
  brand: {
    primaryColor: '#007bff',
    secondaryColor: '#6c757d',
    fontFamily: 'Inter, system-ui, sans-serif',
    logoUrl: '/assets/brand-logo.svg',
    borderRadius: '8px'
  },
  
  // Component-level styling
  components: {
    button: {
      padding: '12px 24px',
      fontSize: '14px',
      fontWeight: '500',
      background: 'var(--embedy-color-primary)',
      border: 'none',
      borderRadius: 'var(--embedy-border-radius)'
    },
    input: {
      border: '1px solid var(--embedy-color-border)',
      borderRadius: 'var(--embedy-border-radius)',
      padding: '10px 12px',
      fontSize: '14px'
    }
  },
  
  // Layout customization
  layout: {
    spacing: 'comfortable', // compact | comfortable | spacious
    direction: 'ltr',
    maxWidth: '600px'
  },
  
  // Dark mode support
  darkMode: {
    enabled: true,
    strategy: 'class', // class | media | manual
    colors: {
      background: '#1a1a1a',
      surface: '#2d2d2d',
      text: '#ffffff'
    }
  }
};
```

### CSS Custom Properties Integration
```css
/* Host application can override any design token */
:root {
  --embedy-color-primary: #your-brand-color;
  --embedy-color-secondary: #6c757d;
  --embedy-border-radius: 4px;
  --embedy-font-family: 'Your Brand Font';
  --embedy-spacing-unit: 8px;
}

/* Automatic dark mode support */
@media (prefers-color-scheme: dark) {
  :root {
    --embedy-color-background: #1a1a1a;
    --embedy-color-text: #ffffff;
  }
}
```

### Runtime Theme Switching
```javascript
// Dynamic theme updates without re-rendering
const embeddedForm = document.querySelector('embedy-invoice-form');
embeddedForm.updateTheme({
  brand: { 
    primaryColor: '#new-color',
    secondaryColor: '#6c757d' 
  },
  components: { 
    button: { 
      borderRadius: '12px',
      backgroundColor: 'var(--embedy-color-primary)',
      color: 'var(--embedy-color-on-primary)'
    } 
  }
});
```

## Schema-Driven Configuration Examples

### Declarative Form Configuration
```json
{
  "$schema": "./form-schema.json",
  "formDefinition": {
    "id": "invoice-form",
    "title": "Invoice Creation",
    "theme": "brand-primary",
    "fields": [
      {
        "id": "client_selection",
        "type": "client-lookup",
        "label": "Client",
        "validation": {
          "required": true,
          "customRules": ["existing_client"]
        },
        "styling": {
          "variant": "outlined",
          "size": "medium"
        }
      }
    ],
    "layout": {
      "type": "grid",
      "columns": 2,
      "gap": "16px"
    },
    "branding": {
      "showLogo": true,
      "customFooter": "Powered by Your Brand"
    }
  }
}
```

### Component Library Configuration
```javascript
// Producer-defined components with consumer theming points
const FormAtoms = {
  // Producers create the component logic and structure
  CurrencyInput: {
    schema: CurrencyInputSchema,
    producerControls: ['validation', 'formatting', 'calculations'],
    consumerTheming: ['colors', 'typography', 'spacing', 'borders'],
    variants: ['outlined', 'filled', 'underlined'],
    sizes: ['small', 'medium', 'large']
  },
  
  // Producers define features, consumers style them
  ClientLookup: {
    schema: ClientLookupSchema,
    producerControls: ['search logic', 'data fetching', 'create flow'],
    consumerTheming: ['colors', 'typography', 'spacing', 'borders', 'shadows'],
    features: ['search', 'create', 'recent', 'favorites'],
    customizations: ['placeholder', 'noResults', 'createPrompt']
  },
  
  // Producers implement business logic, consumers brand the UI
  TaxCalculator: {
    schema: TaxCalculatorSchema,
    producerControls: ['tax rules', 'calculations', 'regional logic'],
    consumerTheming: ['colors', 'typography', 'spacing'],
    capabilities: ['regional', 'multi_rate', 'exemptions', 'compound'],
    display: ['inline', 'tooltip', 'modal', 'sidebar']
  }
};
```

## Integration Examples

### Drop-in Integration
```html
<!-- Minimal setup with default branding -->
<script src="https://cdn.embedy.com/embedy.js"></script>
<embedy-invoice-form api-key="your-api-key"></embedy-invoice-form>
```

### Fully Branded Integration
```javascript
// Complete white-label customization (visual theming only)
import { EmbedyProvider, InvoiceForm } from '@embedy/react';

const App = () => (
  <EmbedyProvider 
    theme={fullBrandTheme}  // Consumer controls all visual aspects
    apiConfig={config}
    whiteLabel={true}>
    <InvoiceForm
      // Note: Headers/footers are styled via theme, not replaced
      // The producer defines the structure, consumer themes it
      headerTheme={customHeaderStyles}
      footerTheme={customFooterStyles}
      onSubmit={handleSubmit} />
  </EmbedyProvider>
);
```

### Enterprise Security Integration
```javascript
// Maximum isolation with custom authentication
<embedy-iframe
  src="/embed/invoice-form"
  sandbox="allow-same-origin allow-scripts allow-forms"
  auth-token="jwt-token"
  csp-nonce="random-nonce"
  theme-url="https://your-domain.com/embedy-theme.css">
</embedy-iframe>
```

## Troubleshooting Code Examples

### Component Loading Debug
```javascript
// Ensure Embedy is properly loaded before use
import('@embedy/core').then(() => {
  document.querySelector('embedy-invoice-form').style.display = 'block';
});

// For web components, check if defined
if (!customElements.get('embedy-invoice-form')) {
  console.error('Embedy component not loaded');
  // Load component definition
  await import('@embedy/components/invoice-form');
}

// For React, ensure proper import
import { InvoiceForm } from '@embedy/react';
// Not: import InvoiceForm from '@embedy/react'; // ❌ Wrong
```

### Styling Debug
```css
/* Ensure proper CSS isolation */
embedy-invoice-form {
  /* Force CSS containment */
  contain: layout style paint;
  isolation: isolate;
}

/* Check for CSS custom property conflicts */
:root {
  /* Use proper variable naming */
  --embedy-color-primary: #007bff; /* ✅ Correct */
  --primary-color: #007bff; /* ❌ May conflict with host */
}

/* Debugging CSS variables */
embedy-invoice-form {
  /* Temporarily override to test */
  --embedy-color-primary: red !important;
}
```

```javascript
// Programmatically debug theming
const component = document.querySelector('embedy-invoice-form');
console.log('Current theme:', component.theme);

// Test theme update
component.updateTheme({
  brand: { primaryColor: '#ff0000' }
}).then(() => {
  console.log('Theme updated successfully');
}).catch(error => {
  console.error('Theme update failed:', error);
});
```

### Navigation Conflict Resolution
```javascript
// Configure navigation isolation
const embedyNav = new EmbedyNavigationManager({
  isolation: {
    stateManagement: 'internal', // Use internal state instead of URL
    conflictResolution: 'namespace' // Namespace all events
  }
});

// Handle navigation conflicts
window.addEventListener('hashchange', (event) => {
  // Only handle embedy navigation
  if (event.newURL.includes('#embedy/')) {
    event.stopPropagation();
    embedyNav.handleHashChange(event);
  }
});

// Alternative: Use postMessage for iframe isolation
if (window.top !== window.self) {
  // We're in an iframe, use postMessage
  embedyNav.config.isolation.stateManagement = 'postMessage';
}
```

### API Integration Debug
```javascript
// Debug API configuration
const apiClient = new EmbedyApiClient({
  baseUrl: 'https://api.yourdomain.com', // ✅ Use full URL
  authToken: 'your-token',
  debug: true // Enable debug logging
});

// Handle CORS issues
// Server-side configuration needed:
/*
Access-Control-Allow-Origin: https://yourdomain.com
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization, X-Embedy-Token
Access-Control-Allow-Credentials: true
*/

// Client-side debugging
fetch('/api/test', {
  method: 'OPTIONS',
  headers: {
    'Content-Type': 'application/json'
  }
}).then(response => {
  console.log('CORS preflight:', response.headers);
}).catch(error => {
  console.error('CORS issue:', error);
});

// Test authentication
apiClient.testAuth().then(() => {
  console.log('Auth working');
}).catch(error => {
  console.error('Auth failed:', error);
  // Check token expiration, refresh if needed
});
```

### Performance Monitoring
```javascript
// Enable performance monitoring
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    if (entry.name.includes('embedy')) {
      console.log(`${entry.name}: ${entry.duration}ms`);
    }
  });
});
observer.observe({ entryTypes: ['measure', 'navigation'] });

// Optimize bundle loading
import('./embedy-core').then(async (core) => {
  // Load only needed components
  if (needsAdvancedValidation) {
    await import('./embedy-validation');
  }
  
  if (isMobile) {
    await import('./embedy-mobile');
  }
});

// Memory leak detection
setInterval(() => {
  const components = document.querySelectorAll('[data-embedy]');
  console.log(`Active components: ${components.length}`);
  
  // Check for memory leaks
  if (performance.memory) {
    console.log('Memory usage:', {
      used: Math.round(performance.memory.usedJSHeapSize / 1048576),
      total: Math.round(performance.memory.totalJSHeapSize / 1048576)
    });
  }
}, 10000);
```

### Browser Compatibility Check
```javascript
// Check for required features
const checkCompatibility = () => {
  const required = {
    customElements: 'customElements' in window,
    shadowDOM: 'attachShadow' in Element.prototype,
    fetch: 'fetch' in window,
    promises: 'Promise' in window,
    webCrypto: 'crypto' in window && 'subtle' in crypto
  };
  
  const missing = Object.entries(required)
    .filter(([key, supported]) => !supported)
    .map(([key]) => key);
  
  if (missing.length > 0) {
    console.warn('Missing browser features:', missing);
    // Load polyfills
    return loadPolyfills(missing);
  }
  
  return Promise.resolve();
};

// Load appropriate polyfills
const loadPolyfills = async (missing) => {
  const polyfills = [];
  
  if (missing.includes('customElements')) {
    polyfills.push(import('@webcomponents/custom-elements'));
  }
  
  if (missing.includes('fetch')) {
    polyfills.push(import('whatwg-fetch'));
  }
  
  await Promise.all(polyfills);
};
```

### CSP Compliance
```javascript
// Detect CSP violations
document.addEventListener('securitypolicyviolation', (event) => {
  if (event.violatedDirective.includes('script-src')) {
    console.error('CSP Script violation:', event);
    // Switch to CSP-compliant mode
    window.EmbedyConfig = {
      cspMode: true,
      inlineStyles: false
    };
  }
});

// CSP-compliant initialization
const initEmbedyCSPMode = () => {
  // Use nonce for scripts
  const scripts = document.querySelectorAll('script[data-embedy]');
  scripts.forEach(script => {
    if (!script.nonce) {
      console.warn('Script missing nonce in CSP mode');
    }
  });
  
  // Use external stylesheets instead of inline styles
  const link = document.createElement('link');
  link.rel = 'stylesheet';
  link.href = '/embedy-styles.css';
  link.nonce = document.querySelector('meta[name="csp-nonce"]')?.content;
  document.head.appendChild(link);
};
```

### Development Debugging Tools
```javascript
// Enable development mode
window.EmbedyConfig = {
  debug: true,
  logLevel: 'verbose',
  showPerformanceMetrics: true
};

// Runtime debugging
const debugEmbedy = () => {
  // List all embedy components
  const components = document.querySelectorAll('[data-embedy], embedy-*');
  console.table(Array.from(components).map(el => ({
    tagName: el.tagName,
    id: el.id,
    theme: el.theme?.brand?.primaryColor,
    isolation: el.isolationLevel,
    ready: el.hasAttribute('data-embedy-ready')
  })));
  
  // Check event listeners
  const events = ['embedy:ready', 'embedy:error', 'embedy:submit'];
  events.forEach(eventType => {
    const listeners = document.querySelectorAll(`[data-${eventType}]`);
    console.log(`${eventType} listeners:`, listeners.length);
  });
};

// Add to global scope for console access
window.debugEmbedy = debugEmbedy;
```

### Error Reporting System
```javascript
// Enhanced error reporting
class EmbedyErrorReporter {
  static report(error, context = {}) {
    const report = {
      error: {
        type: error.type || error.name,
        message: error.message,
        stack: error.stack
      },
      context: {
        userAgent: navigator.userAgent,
        url: window.location.href,
        timestamp: new Date().toISOString(),
        ...context
      },
      embedy: {
        version: window.EmbedyVersion,
        components: this.getActiveComponents(),
        theme: this.getCurrentTheme()
      }
    };
    
    // Send to monitoring service
    if (window.Sentry) {
      window.Sentry.captureException(error, { extra: report });
    }
    
    // Log for development
    if (window.EmbedyConfig?.debug) {
      console.group('🚨 Embedy Error Report');
      console.error('Error:', error);
      console.table(report.context);
      console.log('Full report:', report);
      console.groupEnd();
    }
    
    return report;
  }
  
  static getActiveComponents() {
    return Array.from(document.querySelectorAll('[data-embedy]'))
      .map(el => el.tagName.toLowerCase());
  }
  
  static getCurrentTheme() {
    const component = document.querySelector('[data-embedy]');
    return component?.theme || null;
  }
}

// Auto-attach error reporter
window.addEventListener('error', (event) => {
  if (event.filename?.includes('embedy')) {
    EmbedyErrorReporter.report(event.error, {
      filename: event.filename,
      lineno: event.lineno,
      colno: event.colno
    });
  }
});
```