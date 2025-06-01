# Embedy: Technical Requirements Document

## Executive Summary

This document outlines the technical architecture and implementation requirements for Embedy, a comprehensive toolkit for creating embeddable applications. Based on modern tooling and best practices, this architecture enables deployment across web components, React components, and iframe sandboxes while maintaining consistent functionality and complete brandability.

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
// Base component architecture
import { LitElement, html, css } from 'lit';
import { customElement, property } from 'lit/decorators.js';

@customElement('embedy-base')
export class EmbedyBase extends LitElement {
  @property({ type: Object }) theme = {};
  @property({ type: String }) isolationLevel = 'shadow';
  
  static styles = css`/* Dynamic theme injection */`;
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
**Rationale**: CSS custom properties provide the most flexible and performant theming solution.

**Architecture**:
```css
/* Design token foundation */
:root {
  /* Brand tokens */
  --embedy-color-primary: #007bff;
  --embedy-color-secondary: #6c757d;
  --embedy-color-success: #28a745;
  --embedy-color-danger: #dc3545;
  
  /* Semantic tokens */
  --embedy-button-background: var(--embedy-color-primary);
  --embedy-button-color: white;
  --embedy-button-border-radius: 8px;
  
  /* Spacing system */
  --embedy-space-xs: 4px;
  --embedy-space-sm: 8px;
  --embedy-space-md: 16px;
  --embedy-space-lg: 24px;
  --embedy-space-xl: 32px;
  
  /* Typography scale */
  --embedy-font-family: 'Inter', system-ui, sans-serif;
  --embedy-font-size-xs: 12px;
  --embedy-font-size-sm: 14px;
  --embedy-font-size-md: 16px;
  --embedy-font-size-lg: 18px;
  --embedy-font-size-xl: 20px;
}

/* Dark mode support */
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
// Theme management system
interface ThemeConfig {
  brand: {
    primaryColor: string;
    secondaryColor: string;
    fontFamily: string;
    logoUrl?: string;
    borderRadius: string;
  };
  components: Record<string, CSSProperties>;
  layout: {
    spacing: 'compact' | 'comfortable' | 'spacious';
    direction: 'ltr' | 'rtl';
    maxWidth: string;
  };
  darkMode: {
    enabled: boolean;
    strategy: 'class' | 'media' | 'manual';
    colors: Record<string, string>;
  };
}

class ThemeEngine {
  updateTheme(config: Partial<ThemeConfig>) {
    const root = document.documentElement;
    
    // Update CSS custom properties
    Object.entries(this.flattenTheme(config)).forEach(([key, value]) => {
      root.style.setProperty(`--embedy-${key}`, value);
    });
    
    // Trigger theme change event
    window.dispatchEvent(new CustomEvent('embedy:theme-change', {
      detail: config
    }));
  }
}
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

#### PostMessage Security

**Library Recommendation**: **Postmate** by Dollar Shave Club
**Rationale**: Promise-based API with built-in security validation, minimal bundle size (~1.6KB gzipped), and excellent documentation.

```typescript
// Using Postmate for secure iframe communication
import Postmate from 'postmate';

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
- **Initial Load**: < 50KB gzipped (core framework)
- **Time to Interactive**: < 2 seconds on 3G
- **Component Render**: < 16ms (60fps)
- **Memory Usage**: < 10MB for complete form suite
- **Bundle Analysis**: Automatic size regression detection

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
// Complete TypeScript definitions for all APIs
export interface EmbedyComponent {
  theme: ThemeConfig;
  isolationLevel: IsolationLevel;
  onSubmit?: (data: FormData, validation: ValidationResult) => void;
  onChange?: (fieldId: string, value: unknown) => void;
  onError?: (error: FormError, context: ErrorContext) => void;
}

export interface FormConfig {
  id: string;
  title: string;
  fields: FieldDefinition[];
  layout: LayoutConfig;
  validation: ValidationConfig;
  branding: BrandingConfig;
}
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