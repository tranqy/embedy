# Embedy: Reusable Embeddable Component Toolkit

## Overview

Embedy is a comprehensive toolkit for creating embeddable applications that can be seamlessly integrated into any web environment. The framework provides three distinct embedding methods—web components, React components, and iframes—each optimized for different integration scenarios while maintaining consistent functionality and branding across all deployment contexts.

## Core Purpose

**Universal Embedding**: Deploy the same application logic across web components (native), React components (framework-specific), and sandboxed iframes (maximum isolation).

**Complete Brandability**: Every visual element is customizable through a comprehensive theming system, allowing host applications to maintain their design language.

**Developer-First Experience**: Minimal integration complexity with powerful customization options, supporting everything from simple drop-in components to fully branded white-label solutions.

## Embedding Methods

### Web Components (Native)
```javascript
// Direct DOM integration with shadow DOM isolation
<embedy-invoice-form 
  theme="custom-brand"
  api-endpoint="/api/invoices"
  isolation-level="shadow">
</embedy-invoice-form>
```

**Use Cases:**
- Modern browsers with native web components support
- Maximum performance with minimal bundle size
- Direct DOM integration without framework dependencies
- Shared styling with host application when desired

### React Components
```javascript
// Framework-native integration
import { InvoiceForm } from '@embedy/react';

<InvoiceForm 
  theme={customTheme}
  onSubmit={handleSubmit}
  isolationLevel="component" />
```

**Use Cases:**
- React-based host applications
- Type-safe integration with TypeScript
- Direct access to React ecosystem (hooks, context, etc.)
- Optimal bundle sharing and tree-shaking

### Iframe Sandbox
```javascript
// Maximum security isolation
<embedy-iframe 
  src="/embed/invoice-form"
  sandbox="allow-same-origin allow-scripts"
  theme-url="/themes/custom.css">
</embedy-iframe>
```

**Use Cases:**
- Legacy browser support
- Maximum security isolation requirements
- Third-party hosting with strict CSP policies
- Enterprise environments with security constraints

## Multi-Environment Compatibility

### Environment Detection and Adaptation

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
class FeatureManager {
  async loadOptimalFeatureSet(capabilities) {
    const coreFeatures = await import('./core-forms'); // 15KB
    
    if (capabilities.hasModernBrowser && capabilities.bandwidth > '3G') {
      await import('./advanced-validation'); // +8KB
      await import('./rich-formatting'); // +12KB
    }
    
    if (capabilities.touchDevice) {
      await import('./mobile-optimizations'); // +5KB
    }
    
    return this.assembleFormComponent(loadedFeatures);
  }
}
```

## Comprehensive Theming System

### Multi-Level Customization

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
      background: 'var(--brand-primary)',
      border: 'none',
      borderRadius: 'var(--brand-border-radius)'
    },
    input: {
      border: '1px solid #e1e5e9',
      borderRadius: 'var(--brand-border-radius)',
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
  --embedy-primary: #your-brand-color;
  --embedy-border-radius: 4px;
  --embedy-font-family: 'Your Brand Font';
  --embedy-spacing-unit: 8px;
}

/* Automatic dark mode support */
@media (prefers-color-scheme: dark) {
  :root {
    --embedy-background: #1a1a1a;
    --embedy-text: #ffffff;
  }
}
```

### Runtime Theme Switching

```javascript
// Dynamic theme updates without re-rendering
const embeddedForm = document.querySelector('embedy-invoice-form');
embeddedForm.updateTheme({
  brand: { primaryColor: '#new-color' },
  components: { button: { borderRadius: '12px' } }
});
```

## Schema-Driven Architecture

### Declarative Configuration

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

### Component Library

```javascript
// Atomic design system for consistent composition
const FormAtoms = {
  CurrencyInput: {
    schema: CurrencyInputSchema,
    themeable: ['colors', 'typography', 'spacing', 'borders'],
    variants: ['outlined', 'filled', 'underlined'],
    sizes: ['small', 'medium', 'large']
  },
  
  ClientLookup: {
    schema: ClientLookupSchema,
    themeable: ['colors', 'typography', 'spacing', 'borders', 'shadows'],
    features: ['search', 'create', 'recent', 'favorites'],
    customizations: ['placeholder', 'noResults', 'createPrompt']
  },
  
  TaxCalculator: {
    schema: TaxCalculatorSchema,
    themeable: ['colors', 'typography', 'spacing'],
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
// Complete white-label customization
import { EmbedyProvider, InvoiceForm } from '@embedy/react';

const App = () => (
  <EmbedyProvider 
    theme={fullBrandTheme}
    apiConfig={config}
    whiteLabel={true}>
    <InvoiceForm
      customHeader={<YourBrandHeader />}
      customFooter={<YourBrandFooter />}
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

## Performance Characteristics

**Bundle Sizes:**
- Core framework: 15KB gzipped
- Individual components: 3-8KB each
- Complete form suite: 45KB gzipped
- Theme engine: 5KB gzipped

**Loading Strategy:**
- Progressive enhancement based on capabilities
- Intelligent feature detection and loading
- Automatic fallbacks for legacy environments
- CDN optimization with regional caching

**Runtime Performance:**
- Shadow DOM isolation prevents style conflicts
- Virtual scrolling for large datasets
- Debounced validation and API calls
- Memory-efficient cleanup on unmount