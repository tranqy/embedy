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

## Multi-Target Architecture Examples

### Component Interface Foundation

All Embedy components implement a shared interface to ensure consistency across deployment methods:

```typescript
// core/component-interface.ts
export interface EmbedyComponentInterface {
  // Core functionality that must work in all deployment methods
  validate(): Promise<ValidationResult>;
  submit(): Promise<SubmissionResult>;
  updateTheme(theme: Partial<ThemeConfig>): Promise<void>;
  reset(): void;
  
  // Properties that must be supported across all methods
  theme: ThemeConfig;
  isolationLevel: IsolationLevel;
  disabled: boolean;
  
  // Events that must be emitted consistently
  onValidationChange?: (result: ValidationResult) => void;
  onSubmit?: (data: FormData) => Promise<SubmissionResult>;
  onError?: (error: EmbedyError) => void;
}
```

### Automatic React Wrapper Generation

#### Build-Time Code Generation

React wrappers are automatically generated from Lit components during the build process:

```typescript
// build-tools/react-wrapper-generator.ts
interface ReactWrapperConfig {
  litComponent: string;
  reactComponentName: string;
  props: PropertyDefinition[];
  events: EventDefinition[];
}

class ReactWrapperGenerator {
  generateWrapper(config: ReactWrapperConfig): string {
    return `
import React, { useRef, useEffect, forwardRef } from 'react';
import { ${config.litComponent} } from '../web-components/${config.litComponent}';

// Ensure web component is registered
if (!customElements.get('${kebabCase(config.litComponent)}')) {
  customElements.define('${kebabCase(config.litComponent)}', ${config.litComponent});
}

export interface ${config.reactComponentName}Props {
  ${this.generatePropTypes(config.props)}
  ${this.generateEventProps(config.events)}
}

export const ${config.reactComponentName} = forwardRef<
  ${config.litComponent},
  ${config.reactComponentName}Props
>((props, ref) => {
  const elementRef = useRef<${config.litComponent}>(null);
  
  // Forward ref to the actual web component
  useEffect(() => {
    if (ref && elementRef.current) {
      if (typeof ref === 'function') {
        ref(elementRef.current);
      } else {
        ref.current = elementRef.current;
      }
    }
  }, [ref]);
  
  // Sync props to web component properties
  useEffect(() => {
    const element = elementRef.current;
    if (element) {
      ${this.generatePropSync(config.props)}
    }
  }, [${config.props.map(p => `props.${p.name}`).join(', ')}]);
  
  // Setup event listeners
  useEffect(() => {
    const element = elementRef.current;
    if (element) {
      ${this.generateEventListeners(config.events)}
      
      return () => {
        ${this.generateEventCleanup(config.events)}
      };
    }
  }, [${config.events.map(e => `props.${e.reactPropName}`).join(', ')}]);
  
  return (
    <${kebabCase(config.litComponent)}
      ref={elementRef}
      {...this.filterDOMProps(props)}
    />
  );
});
`;
  }
  
  private generatePropSync(props: PropertyDefinition[]): string {
    return props.map(prop => 
      `element.${prop.name} = props.${prop.name};`
    ).join('\n      ');
  }
  
  private generateEventListeners(events: EventDefinition[]): string {
    return events.map(event => `
      const ${event.handlerName} = (e: CustomEvent) => {
        if (props.${event.reactPropName}) {
          props.${event.reactPropName}(e.detail);
        }
      };
      element.addEventListener('${event.nativeName}', ${event.handlerName});`
    ).join('\n');
  }
}
```

#### Type-Safe Property Decoration

The `@embedyProperty` decorator captures type information for React wrapper generation:

```typescript
// core/embedy-property.ts
export function embedyProperty<T>(options?: PropertyOptions) {
  return function(target: any, propertyKey: string) {
    // Store type information for React wrapper generation
    const existingProps = Reflect.getMetadata('embedy:props', target) || [];
    existingProps.push({
      name: propertyKey,
      type: Reflect.getMetadata('design:type', target, propertyKey),
      options
    });
    Reflect.setMetadata('embedy:props', existingProps, target);
    
    // Apply Lit property decorator
    return property(options)(target, propertyKey);
  };
}

// Enhanced component definition with automatic type inference
export function defineEmbedyComponent<T extends BaseComponentProps>(
  tagName: string,
  componentClass: Constructor<LitElement>
) {
  // Extract prop types from the component class
  const propTypes = Reflect.getMetadata('embedy:props', componentClass.prototype);
  
  // Generate TypeScript definitions for React wrapper
  generateReactTypes(tagName, propTypes);
  
  // Register web component
  customElements.define(tagName, componentClass);
  
  return componentClass as Constructor<LitElement & T>;
}
```

### Feature Parity Management

#### Capability Detection System

Different deployment methods have different capabilities that need to be detected and adapted:

```typescript
// core/feature-capability-manager.ts
export class FeatureCapabilityManager {
  private capabilities = new Map<string, boolean>();
  
  constructor(deploymentMethod: 'web-component' | 'react' | 'iframe') {
    this.detectCapabilities(deploymentMethod);
  }
  
  private detectCapabilities(method: string) {
    switch (method) {
      case 'web-component':
        this.capabilities.set('directDOMAccess', true);
        this.capabilities.set('shadowDOM', true);
        this.capabilities.set('customEvents', true);
        this.capabilities.set('cssCustomProperties', true);
        break;
      case 'react':
        this.capabilities.set('reactHooks', true);
        this.capabilities.set('reactContext', true);
        this.capabilities.set('jsxProps', true);
        this.capabilities.set('stateManagement', true);
        break;
      case 'iframe':
        this.capabilities.set('postMessage', true);
        this.capabilities.set('sandboxSecurity', true);
        this.capabilities.set('crossOrigin', true);
        this.capabilities.set('isolatedContext', true);
        break;
    }
  }
  
  // Adapt features based on deployment method capabilities
  adaptFeature(featureName: string, implementations: FeatureImplementations) {
    if (this.capabilities.get('reactHooks') && implementations.react) {
      return implementations.react;
    } else if (this.capabilities.get('postMessage') && implementations.iframe) {
      return implementations.iframe;
    } else {
      return implementations.webComponent;
    }
  }
  
  // Check if a specific capability is available
  hasCapability(capability: string): boolean {
    return this.capabilities.get(capability) || false;
  }
}

// Feature implementation variants
interface FeatureImplementations {
  webComponent: () => any;
  react?: () => any;
  iframe?: () => any;
}

// Example usage
const themeManager = capabilityManager.adaptFeature('themeManagement', {
  webComponent: () => new CSSCustomPropertyThemeManager(),
  react: () => new ReactContextThemeManager(),
  iframe: () => new PostMessageThemeManager()
});
```

### React Adaptation Layer

For features that can't be directly translated to React patterns, we use adaptation layers:

```typescript
// adapters/react-adaptation-layer.ts
export class ReactAdaptationLayer {
  // Handle shadow DOM in React (where shadow DOM isn't native)
  static adaptShadowDOM(Component: React.ComponentType) {
    return React.forwardRef((props, ref) => {
      const containerRef = useRef<HTMLDivElement>(null);
      const shadowRef = useRef<ShadowRoot | null>(null);
      
      useEffect(() => {
        if (containerRef.current && !shadowRef.current) {
          // Create shadow DOM for React component
          shadowRef.current = containerRef.current.attachShadow({ mode: 'open' });
          
          // Inject styles into shadow DOM
          const styleSheet = new CSSStyleSheet();
          styleSheet.replaceSync(getComponentStyles());
          shadowRef.current.adoptedStyleSheets = [styleSheet];
        }
      }, []);
      
      return (
        <div ref={containerRef}>
          {/* Render React component inside shadow DOM */}
          {shadowRef.current && 
            ReactDOM.createPortal(<Component {...props} ref={ref} />, shadowRef.current)
          }
        </div>
      );
    });
  }
  
  // Handle custom events in React
  static adaptCustomEvents(webComponentRef: RefObject<HTMLElement>, eventMap: EventMap) {
    useEffect(() => {
      const element = webComponentRef.current;
      if (!element) return;
      
      const eventHandlers = new Map();
      
      Object.entries(eventMap).forEach(([eventName, reactHandler]) => {
        const handler = (event: CustomEvent) => {
          reactHandler(event.detail);
        };
        
        element.addEventListener(eventName, handler);
        eventHandlers.set(eventName, handler);
      });
      
      return () => {
        eventHandlers.forEach((handler, eventName) => {
          element?.removeEventListener(eventName, handler);
        });
      };
    }, [webComponentRef.current, eventMap]);
  }
  
  // Translate Lit reactive properties to React state
  static adaptReactiveProperties<T extends Record<string, any>>(
    initialProps: T,
    webComponentRef: RefObject<HTMLElement & T>
  ): [T, (updates: Partial<T>) => void] {
    const [state, setState] = useState<T>(initialProps);
    
    const updateProperties = useCallback((updates: Partial<T>) => {
      setState(prev => ({ ...prev, ...updates }));
      
      // Sync to web component
      if (webComponentRef.current) {
        Object.entries(updates).forEach(([key, value]) => {
          (webComponentRef.current as any)[key] = value;
        });
      }
    }, [webComponentRef]);
    
    return [state, updateProperties];
  }
}
```

### Type System Integration

#### Shared Type Definitions

Types are shared across all deployment methods through a common type definition system:

```typescript
// types/shared-component-types.ts
export interface BaseComponentProps {
  theme?: Partial<ThemeConfig>;
  isolationLevel?: IsolationLevel;
  disabled?: boolean;
  className?: string;
  id?: string;
}

export interface FormComponentProps extends BaseComponentProps {
  validationRules?: ValidationConfig;
  onSubmit?: (data: FormData) => Promise<SubmissionResult>;
  onChange?: (fieldId: string, value: unknown) => void;
  onValidation?: (result: ValidationResult) => void;
}

// Utility type to extract props from web component
export type ExtractComponentProps<T> = T extends LitElement 
  ? { [K in keyof T]: T[K] extends Function ? T[K] : T[K] }
  : never;

// Auto-generate React prop types from web component
export type ReactPropsFromWebComponent<T extends LitElement> = 
  ExtractComponentProps<T> & {
    // Add React-specific props
    children?: React.ReactNode;
    className?: string;
    style?: React.CSSProperties;
  };
```

#### Complete Component Implementation Example

Here's how a complete component is implemented with multi-target support:

```typescript
// components/invoice-form/invoice-form.ts
@defineEmbedyComponent<InvoiceFormProps>('embedy-invoice-form', InvoiceFormComponent)
export class InvoiceFormComponent extends EmbedyBase implements EmbedyComponentInterface {
  @embedyProperty({ type: String })
  apiEndpoint: string = '';
  
  @embedyProperty({ type: Object })
  validationRules: ValidationConfig = {};
  
  @embedyProperty({ type: Function })
  onSubmit?: (data: FormData) => Promise<SubmissionResult>;
  
  async validate(): Promise<ValidationResult> {
    // Implementation that works across all deployment methods
    const result = await this.validator.validate(this.getFormData());
    this.dispatchEvent(new CustomEvent('embedy:validation', { detail: result }));
    return result;
  }
  
  async submit(): Promise<SubmissionResult> {
    const validationResult = await this.validate();
    if (!validationResult.valid) {
      throw new EmbedyError('VALIDATION_FAILED', { errors: validationResult.errors });
    }
    
    const formData = this.getFormData();
    if (this.onSubmit) {
      return await this.onSubmit(formData);
    }
    
    // Default submission logic
    return await this.apiClient.submitForm(formData);
  }
  
  render() {
    return html`
      <div class="invoice-form">
        <!-- Component template that works in all contexts -->
      </div>
    `;
  }
}

// Auto-generated React wrapper (created during build)
export const InvoiceForm = ReactAdaptationLayer.adaptShadowDOM(
  React.forwardRef<HTMLElement, InvoiceFormProps>((props, ref) => {
    const webComponentRef = useRef<InvoiceFormComponent>(null);
    
    // Adapt custom events to React callbacks
    ReactAdaptationLayer.adaptCustomEvents(webComponentRef, {
      'embedy:submit': props.onSubmit,
      'embedy:validation': props.onValidation,
      'embedy:error': props.onError
    });
    
    // Adapt reactive properties
    const [componentState, updateComponent] = ReactAdaptationLayer.adaptReactiveProperties(
      { 
        theme: props.theme, 
        disabled: props.disabled,
        validationRules: props.validationRules 
      },
      webComponentRef
    );
    
    return (
      <embedy-invoice-form
        ref={mergeRefs([ref, webComponentRef])}
        api-endpoint={props.apiEndpoint}
        {...filterProps(props)}
      />
    );
  })
);

// TypeScript interface for React component (auto-generated)
export interface InvoiceFormProps extends FormComponentProps {
  apiEndpoint?: string;
  validationRules?: ValidationConfig;
}
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
## Security Implementation Examples

### Origin Validation for PostMessage Communication

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
  
  validateOrigin(origin: string, messageType?: string): ValidationResult {
    if (\!origin) {
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
    // Implementation validates subdomains against trusted base domains
    // Prevents subdomain takeover attempts
    // Returns validation result with risk assessment
  }
  
  private normalizeOrigin(origin: string): string {
    // Normalizes origin URLs to consistent format
    // Handles protocol, hostname, and port normalization
  }
}
```

### Encryption Key Management

```typescript
// security/crypto-manager.ts
class EmbedyCryptoManager {
  private config: CryptoConfig;
  private keyCache = new Map<string, CryptoKey>();
  private sessionSalt: Uint8Array;
  
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
  
  // Secure key rotation
  async rotateKeys(): Promise<void> {
    // Generate new session salt
    this.sessionSalt = crypto.getRandomValues(new Uint8Array(this.config.saltLength));
    // Clear key cache to force regeneration
    this.keyCache.clear();
  }
}
```

### CSP Nonce Implementation

```typescript
// security/csp-manager.ts
class CSPManager {
  private currentNonce: string | null = null;
  private config: CSPConfig;
  
  // Create CSP-compliant script elements
  createScript(src?: string, textContent?: string): HTMLScriptElement {
    const script = document.createElement('script');
    
    if (this.currentNonce) {
      script.nonce = this.currentNonce;
    }
    
    if (src) {
      // Validate source against trusted domains
      if (this.config.strictMode && \!this.isTrustedSource(src)) {
        throw new CSPError('Script source not in trusted domains', { src });
      }
      script.src = src;
    }
    
    if (textContent) {
      if (this.config.strictMode && \!this.currentNonce) {
        throw new CSPError('Inline scripts require nonce in strict mode');
      }
      script.textContent = textContent;
    }
    
    return script;
  }
  
  // Apply styles in CSP-compliant way
  applyStyles(element: HTMLElement, styles: Record<string, string>): void {
    if (this.config.allowUnsafeInline) {
      // Direct style application when unsafe-inline is allowed
      Object.assign(element.style, styles);
    } else {
      // Use CSS custom properties for CSP compliance
      Object.entries(styles).forEach(([property, value]) => {
        const customProperty = `--embedy-${property.replace(/([A-Z])/g, '-$1').toLowerCase()}`;
        element.style.setProperty(customProperty, value);
      });
      element.classList.add('embedy-dynamic-styles');
    }
  }
  
  // Validate current CSP compliance
  validateCSPCompliance(): CSPValidationResult {
    const violations = [];
    
    // Check for nonce availability
    if (this.config.strictMode && \!this.currentNonce) {
      violations.push({
        type: 'MISSING_NONCE',
        severity: 'HIGH',
        message: 'No CSP nonce available in strict mode'
      });
    }
    
    // Check for unsafe inline styles and scripts
    // Returns compliance score and detailed violations
    
    return {
      compliant: violations.length === 0,
      violations,
      score: this.calculateComplianceScore(violations)
    };
  }
}
```

### Clickjacking Protection

```typescript
// security/clickjacking-protection.ts
class ClickjackingProtection {
  private config: ClickjackingConfig;
  private isEmbedded: boolean;
  private parentOrigin: string | null = null;
  
  private validateEmbeddingContext(): void {
    try {
      // Get parent origin from referrer
      this.parentOrigin = document.referrer ? new URL(document.referrer).origin : null;
      
      if (\!this.parentOrigin) {
        throw new ClickjackingError('Cannot determine parent origin');
      }
      
      // Validate against allowed parents
      if (\!this.isAllowedParent(this.parentOrigin)) {
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
      }
    }
  }
  
  private addVisualIndicators(): void {
    // Add clear visual boundary with embedded content indicator
    const indicator = document.createElement('div');
    indicator.className = 'embedy-embedded-indicator';
    
    // Style using adoptedStyleSheets for CSP compliance
    const styleSheet = new CSSStyleSheet();
    styleSheet.replaceSync(embedIndicatorStyles);
    document.adoptedStyleSheets = [...document.adoptedStyleSheets, styleSheet];
    
    // Add indicator to page
    document.body.insertBefore(indicator, document.body.firstChild);
  }
  
  private setupParentCommunication(): void {
    // Send handshake to parent for additional validation
    if (window.parent && this.parentOrigin) {
      const handshake = {
        type: 'embedy-handshake',
        origin: window.location.origin,
        timestamp: Date.now(),
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
}
```
EOF < /dev/null
## Build System and Progressive Loading Examples

### Progressive Feature Loading - Device Capabilities Detection

```typescript
// core/progressive-loader.ts
interface DeviceCapabilities {
  bandwidth: '2G' | '3G' | '4G' | 'wifi';
  memory: number; // GB
  touchDevice: boolean;
  screenSize: 'small' | 'medium' | 'large';
  hasModernBrowser: boolean;
  supportsWebComponents: boolean;
  supportsIntersectionObserver: boolean;
}

class ProgressiveFeatureLoader {
  private capabilities: DeviceCapabilities;
  private loadedFeatures = new Set<string>();
  
  constructor() {
    this.capabilities = this.detectCapabilities();
  }
  
  private detectCapabilities(): DeviceCapabilities {
    return {
      bandwidth: this.detectBandwidth(),
      memory: this.detectMemory(),
      touchDevice: 'ontouchstart' in window,
      screenSize: this.detectScreenSize(),
      hasModernBrowser: this.detectModernBrowser(),
      supportsWebComponents: 'customElements' in window,
      supportsIntersectionObserver: 'IntersectionObserver' in window
    };
  }
  
  async loadOptimalFeatureSet(): Promise<ComponentLibrary> {
    // Core features (always loaded - 18KB gzipped)
    const core = await import('./core/forms');
    
    // Progressive enhancement based on capabilities
    const features = await Promise.allSettled([
      this.loadAdvancedValidation(),
      this.loadMobileOptimizations(),
      this.loadVirtualScrolling(),
      this.loadRichFormatting(),
      this.loadAccessibilityFeatures()
    ]);
    
    return this.assembleLibrary(core, features.filter(this.isSuccessful));
  }
  
  private async loadAdvancedValidation(): Promise<any> {
    if (this.capabilities.hasModernBrowser && 
        this.capabilities.bandwidth \!== '2G' &&
        this.capabilities.memory >= 2) {
      return import('./features/advanced-validation'); // +8KB
    }
    return null;
  }
}
```

### Bundle Splitting Configuration

```typescript
// vite.config.ts - Enhanced configuration
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        // Manual chunk splitting for optimal caching
        manualChunks: (id) => {
          // Vendor libraries (rarely change, long cache)
          if (id.includes('node_modules')) {
            if (id.includes('lit')) return 'vendor-lit';
            if (id.includes('react')) return 'vendor-react';
            if (id.includes('zod')) return 'vendor-validation';
            if (id.includes('postmate')) return 'vendor-iframe';
            return 'vendor-misc';
          }
          
          // Feature chunks (loaded conditionally)
          if (id.includes('src/features/')) {
            const feature = id.split('src/features/')[1].split('/')[0];
            return `feature-${feature}`;
          }
          
          // Component chunks (loaded on demand)
          if (id.includes('src/components/')) {
            const component = id.split('src/components/')[1].split('/')[0];
            return `component-${component}`;
          }
          
          // Core functionality (always loaded)
          if (id.includes('src/core/')) {
            return 'core';
          }
        }
      }
    }
  }
});
```

### Polyfill Loader with Feature Detection

```typescript
// core/polyfill-loader.ts
class PolyfillLoader {
  private static readonly FEATURE_TESTS = {
    'custom-elements': () => 'customElements' in window,
    'shadow-dom': () => 'attachShadow' in Element.prototype,
    'es6-modules': () => 'noModule' in HTMLScriptElement.prototype,
    'fetch': () => 'fetch' in window,
    'promises': () => 'Promise' in window,
    'web-crypto': () => 'crypto' in window && 'subtle' in crypto,
    'intersection-observer': () => 'IntersectionObserver' in window,
    'resize-observer': () => 'ResizeObserver' in window,
    'css-custom-properties': () => window.CSS && CSS.supports('color', 'var(--test)'),
    'css-containment': () => window.CSS && CSS.supports('contain', 'layout')
  };
  
  static async loadRequiredPolyfills(config: PolyfillConfig = {
    features: ['custom-elements', 'shadow-dom', 'fetch', 'promises'],
    loadStrategy: 'eager',
    fallbackTimeout: 5000
  }): Promise<void> {
    const missingFeatures = this.detectMissingFeatures(config.features);
    
    if (missingFeatures.length === 0) {
      return; // All features are natively supported
    }
    
    switch (config.loadStrategy) {
      case 'eager':
        await this.loadPolyfillsEager(missingFeatures, config.fallbackTimeout);
        break;
      case 'lazy':
        await this.loadPolyfillsLazy(missingFeatures);
        break;
      case 'on-demand':
        this.setupOnDemandPolyfills(missingFeatures);
        break;
    }
  }
}
```

### Fallback Manager Implementation

```typescript
// core/fallback-manager.ts
class FallbackManager {
  static async loadWithFallback<T>(
    primaryImport: () => Promise<T>,
    fallbackStrategies: Array<() => Promise<T>>,
    config: Partial<FallbackConfig> = {}
  ): Promise<T> {
    const fullConfig = { ...this.DEFAULT_CONFIG, ...config };
    
    // Try primary import with retries
    try {
      return await this.retryImport(primaryImport, fullConfig.retryAttempts, fullConfig.retryDelay);
    } catch (primaryError) {
      console.warn('Primary import failed:', primaryError);
    }
    
    // Try fallback strategies in order
    for (let i = 0; i < fallbackStrategies.length; i++) {
      try {
        console.info(`Attempting fallback strategy ${i + 1}/${fallbackStrategies.length}`);
        return await this.retryImport(fallbackStrategies[i], 2, fullConfig.retryDelay);
      } catch (fallbackError) {
        console.warn(`Fallback strategy ${i + 1} failed:`, fallbackError);
      }
    }
    
    // All strategies failed - provide graceful degradation
    if (fullConfig.gracefulDegradation) {
      return this.createGracefulFallback();
    } else {
      throw new Error('All import strategies failed');
    }
  }
  
  private static createGracefulFallback<T>(): T {
    // Return a minimal implementation that prevents crashes
    return {
      initialize: () => Promise.resolve(),
      render: () => '<div class="embedy-fallback">Component temporarily unavailable</div>',
      destroy: () => {},
      updateTheme: () => Promise.resolve(),
      validate: () => Promise.resolve({ valid: true, errors: [] }),
      submit: () => Promise.reject(new Error('Feature unavailable'))
    } as T;
  }
}
```

### Network-Aware Loading

```typescript
// Network-aware loading implementation
class NetworkAwareLoader {
  private isOnline = navigator.onLine;
  private connectionQuality = this.detectConnectionQuality();
  
  constructor() {
    window.addEventListener('online', () => this.isOnline = true);
    window.addEventListener('offline', () => this.isOnline = false);
  }
  
  async loadComponent(componentName: string): Promise<any> {
    if (\!this.isOnline) {
      // Try to load from cache or local storage
      return this.loadFromCache(componentName);
    }
    
    if (this.connectionQuality === 'slow') {
      // Load minimal version for slow connections
      return this.loadMinimalVersion(componentName);
    }
    
    // Normal loading with fallbacks
    return FallbackManager.loadWithFallback(
      () => import(`./components/${componentName}`),
      FallbackManager.createComponentFallbacks(componentName)
    );
  }
  
  private createOfflineFallback(componentName: string) {
    return {
      render: () => `<div class="embedy-offline">
        <h3>${componentName}</h3>
        <p>This component is temporarily unavailable while offline.</p>
        <button onclick="window.location.reload()">Retry when online</button>
      </div>`
    };
  }
  
  private detectConnectionQuality(): 'fast' | 'slow' {
    const connection = (navigator as any).connection;
    if (connection) {
      return connection.effectiveType === '2G' || connection.effectiveType === 'slow-2g' ? 'slow' : 'fast';
    }
    return 'fast';
  }
}
```

### Application Bootstrap Example

```typescript
// Application initialization with progressive loading
async function initializeEmbedy() {
  try {
    // Load required polyfills first
    await PolyfillLoader.loadRequiredPolyfills({
      features: ['custom-elements', 'shadow-dom', 'fetch', 'intersection-observer'],
      loadStrategy: 'eager',
      fallbackTimeout: 5000
    });
    
    // Initialize progressive loader
    const progressiveLoader = new ProgressiveFeatureLoader();
    const componentLibrary = await progressiveLoader.loadOptimalFeatureSet();
    
    // Initialize core with loaded features
    const { EmbedyCore } = await import('./core/embedy-core');
    await EmbedyCore.initialize(componentLibrary);
    
    console.log('Embedy initialized successfully');
    
  } catch (error) {
    console.error('Failed to initialize Embedy:', error);
    
    // Load minimal fallback implementation
    const fallbackLibrary = await import('./fallbacks/basic-forms');
    await fallbackLibrary.initialize();
  }
}
```
EOF < /dev/null