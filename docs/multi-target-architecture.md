# Multi-Target Architecture Implementation

This document provides detailed implementation guidance for Embedy's multi-target component system that enables deployment across web components, React components, and iframe sandboxes.

## Component Interface Foundation

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

## Automatic React Wrapper Generation

### Build-Time Code Generation

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

### Type-Safe Property Decoration

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

## Feature Parity Management

### Capability Detection System

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

## React Adaptation Layer

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

## Type System Integration

### Shared Type Definitions

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

### Usage Example

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

This architecture ensures that:

1. **Type Safety**: Full TypeScript support across all deployment methods
2. **Feature Parity**: Core functionality works identically in all contexts
3. **Framework Integration**: Natural patterns for each framework (React hooks, web component properties, iframe postMessage)
4. **Automatic Generation**: Minimal manual work to maintain React wrappers
5. **Adaptation**: Graceful handling of framework-specific features

The system automatically detects capabilities and adapts features accordingly, ensuring a consistent developer experience regardless of the chosen deployment method.