# Embedy: Reusable Embeddable Component Toolkit

## Overview

Embedy is a comprehensive toolkit for creating embeddable applications that can be seamlessly integrated into any web environment. The framework provides three distinct embedding methods—web components, React components, and iframes—each optimized for different integration scenarios while maintaining consistent functionality and branding across all deployment contexts.

## Core Purpose

**Universal Embedding**: Deploy the same application logic across web components (native), React components (framework-specific), and sandboxed iframes (maximum isolation).

**Complete Brandability**: Every visual element is customizable through a comprehensive theming system, allowing host applications to maintain their design language.

**Developer-First Experience**: Minimal integration complexity with powerful customization options, supporting everything from simple drop-in components to fully branded white-label solutions.

## Architecture: Producer vs Consumer Control

### What Producers Control (Embedy App Developers)
- **Component Composition**: Build custom flows, screens, and component hierarchies
- **Flow Logic**: Define business logic, validation rules, and data processing
- **Structural Components**: Create and control headers, footers, navigation, and layout structures
- **Feature Implementation**: Bring their own components and integrate custom functionality
- **Data Flow**: Manage state, API integrations, and form submissions
- **Base Functionality**: Define the core behavior and capabilities of the embedded application

### What Consumers Control (Host Applications)
- **Visual Theming**: Extensive branding through colors, typography, spacing, and visual styles
- **Design Tokens**: Override any CSS custom property to match brand guidelines
- **Component Styling**: Customize the appearance of all visual elements
- **Dark Mode**: Configure light/dark themes and automatic switching
- **Responsive Behavior**: Adjust layouts and spacing for different screen sizes
- **Localization**: Provide translations and regional formatting

### Key Architectural Principle
Embedy follows a clear separation of concerns: **Producers define structure and behavior, Consumers define appearance and branding**. This ensures that embedded applications maintain their functional integrity while seamlessly adopting the host's visual identity.

## Embedding Methods

### Web Components (Native)
The web components integration method provides direct DOM integration with shadow DOM isolation. Components are defined using custom element syntax with configurable theme, API endpoint, and isolation level properties.

**Use Cases:**
- Modern browsers with native web components support
- Maximum performance with minimal bundle size
- Direct DOM integration without framework dependencies
- Shared styling with host application when desired

**Key Features:**
- Shadow DOM boundary prevents style conflicts
- Custom element registration with browser standards
- Theme property binding for dynamic styling
- Configurable isolation levels

*[See Web Components examples in code-examples.md](./code-examples.md#web-components-native)*

### React Components
The React integration provides framework-native components that integrate seamlessly with React applications. Components use standard React patterns including hooks, context, and TypeScript interfaces.

**Use Cases:**
- React-based host applications
- Type-safe integration with TypeScript
- Direct access to React ecosystem (hooks, context, etc.)
- Optimal bundle sharing and tree-shaking

**Key Features:**
- React Hook Form integration for validation
- TypeScript interfaces for type safety
- Provider pattern for configuration
- Error boundary support

*[See React Components examples in code-examples.md](./code-examples.md#react-components)*

### Iframe Sandbox
The iframe integration method provides maximum security isolation by running the embedded application in a separate browsing context with configurable sandbox restrictions.

**Use Cases:**
- Legacy browser support
- Maximum security isolation requirements
- Third-party hosting with strict CSP policies
- Enterprise environments with security constraints

**Key Features:**
- Sandbox attribute configuration
- PostMessage communication
- Cross-origin theme injection
- Authentication token passing

*[See Iframe Sandbox examples in code-examples.md](./code-examples.md#iframe-sandbox)*

## Multi-Environment Compatibility

### Environment Detection and Adaptation

Embedy automatically detects the host environment capabilities and constraints to optimize integration. The detection system analyzes framework presence, security policies, performance budgets, and navigation systems to determine the best configuration.

**Framework Detection**: Identifies React, Angular, Vue, or legacy environments to optimize component loading and integration patterns.

**Constraint Analysis**: Evaluates CSP restrictions, performance budgets, mobile environments, and accessibility requirements to adapt behavior.

**Navigation Conflict Detection**: Scans for existing navigation systems, URL routing, modal management, and z-index usage to prevent conflicts.

*[See Environment Detection examples in code-examples.md](./code-examples.md#environment-detection-and-adaptation)*

### Adaptive Security Model

The security adapter dynamically determines the appropriate isolation level based on host environment constraints. The system supports three levels of isolation, each optimized for different security requirements.

**Isolation Levels:**
- **IFRAME_SANDBOX**: Maximum isolation for strict CSP or payment environments
- **SHADOW_DOM_ISOLATED**: Style and DOM isolation for privacy requirements
- **COMPONENT_BOUNDARY**: Standard component model for normal integration

**Security Features:**
- Automatic CSP compliance detection
- Payment processing environment adaptation
- Data privacy requirement assessment
- Cross-origin communication validation

*[See Adaptive Security examples in code-examples.md](./code-examples.md#adaptive-security-model)*

### Progressive Feature Loading

The feature management system loads components and capabilities based on browser support, network conditions, and device characteristics. This ensures optimal performance while maintaining functionality across all environments.

**Loading Strategy:**
- Core features always loaded (18KB with polyfills)
- Advanced features conditionally loaded based on capabilities
- Mobile optimizations for touch devices
- Automatic fallbacks for legacy environments

**Performance Optimization:**
- Bandwidth-aware feature loading
- Browser capability detection
- Touch device optimizations
- Progressive enhancement patterns

*[See Progressive Loading examples in code-examples.md](./code-examples.md#progressive-feature-loading)*

## Navigation Isolation

### Core Navigation Principle
Embedy applications maintain independent navigation systems that are visually distinct and functionally isolated from the host application. This prevents conflicts between host and embedded navigation while ensuring a consistent user experience.

### Navigation Patterns

**Embedded Menu Navigation**: A dropdown or overlay menu system that provides navigation within the embedded component without affecting the host application's navigation state.

**Tab Navigation**: Horizontal tab interface for multi-step forms or applications, with visual boundaries and scoped state management.

**Stepper Navigation**: Linear progression interface for wizard-style flows, typically self-contained with minimal visual boundaries.

**Sidebar Navigation**: Vertical menu system for complex applications requiring dedicated navigation space.

**Key Isolation Features:**
- Visual boundary enforcement
- Namespaced event handling
- Scoped state management
- Conflict resolution mechanisms

*[See Navigation Configuration examples in code-examples.md](./code-examples.md#navigation-isolation-examples)*

### Visual Boundary Implementation

Navigation components implement clear visual separation through CSS containment, isolation properties, and customizable borders. The visual boundaries are consumer-themable while maintaining producer-defined structure.

**Boundary Features:**
- CSS containment for layout and style isolation
- Configurable borders, backgrounds, and shadows
- Mobile-responsive positioning
- Host style inheritance prevention

*[See Visual Boundary CSS in code-examples.md](./code-examples.md#visual-boundary-css)*

### Navigation State Management

The navigation manager provides isolated state handling with multiple strategies for different integration scenarios. State can be managed internally, through URL hash routing, or via postMessage communication.

**State Management Options:**
- Internal state for component-only navigation
- URL hash routing with namespace prefixing
- PostMessage communication for iframe isolation
- Event-based coordination with host applications

*[See Navigation State Management in code-examples.md](./code-examples.md#navigation-state-management)*

## Comprehensive Theming System

### Multi-Level Customization

The theming system provides hierarchical customization from brand identity through component-level styling to layout configuration. All theming is consumer-controlled while maintaining producer-defined functionality.

**Theming Hierarchy:**
- **Brand Identity**: Primary colors, typography, logos, and border radius
- **Component Styling**: Individual component appearance and behavior
- **Layout Configuration**: Spacing, direction, and container sizing
- **Dark Mode Support**: Automatic and manual theme switching

**Customization Scope:**
- CSS custom properties for all visual elements
- Runtime theme switching without re-rendering
- Responsive design token adaptation
- Cross-component consistency enforcement

*[See Theme Configuration examples in code-examples.md](./code-examples.md#multi-level-theme-configuration)*

### CSS Custom Properties Integration

The theming foundation uses CSS custom properties with a standardized naming convention. Host applications can override any design token to match their brand guidelines while maintaining functional integrity.

**Variable Naming Convention**: All Embedy variables use the `--embedy-*` prefix to prevent conflicts with host application styles.

**Theme Token Categories:**
- Color tokens for all UI elements
- Spacing tokens for consistent layout
- Typography tokens for font configuration
- Border and shadow tokens for visual effects

**Dark Mode Support**: Automatic detection and manual override capabilities with media query integration.

*[See CSS Custom Properties in code-examples.md](./code-examples.md#css-custom-properties-integration)*

### Runtime Theme Switching

The theme engine supports dynamic updates without component re-rendering. Theme changes are applied immediately through CSS custom property updates with event notification.

**Dynamic Capabilities:**
- Live theme preview and testing
- Gradual theme transitions
- Component-specific overrides
- Theme validation and error handling

*[See Runtime Theme examples in code-examples.md](./code-examples.md#runtime-theme-switching)*

## Schema-Driven Architecture

### Declarative Configuration

Embedy uses JSON schemas to define form structure, validation rules, and component behavior. This approach enables declarative configuration with TypeScript inference and runtime validation.

**Schema Benefits:**
- Type-safe configuration with IDE support
- Runtime validation and error reporting
- Version-controlled form definitions
- Cross-platform configuration sharing

**Configuration Areas:**
- Form field definitions and validation
- Layout and presentation configuration
- Branding and visual customization
- Integration and API configuration

*[See Declarative Configuration in code-examples.md](./code-examples.md#declarative-form-configuration)*

### Component Library Architecture

The component library follows atomic design principles with clear separation between producer-controlled functionality and consumer-controlled styling. Each component exposes specific theming points while maintaining encapsulated behavior.

**Component Structure:**
- **Producer Controls**: Validation, business logic, data processing, and API integration
- **Consumer Theming**: Colors, typography, spacing, borders, and visual effects
- **Variant System**: Predefined styling variants (outlined, filled, underlined)
- **Size System**: Consistent sizing across components (small, medium, large)

**Customization Points**: Each component provides specific styling interfaces while protecting functional implementation.

*[See Component Library examples in code-examples.md](./code-examples.md#component-library-configuration)*

## Integration Examples

### Drop-in Integration
The simplest integration method requires only script inclusion and HTML element placement. This approach uses default branding and configuration for rapid prototyping or minimal customization needs.

**Characteristics:**
- Single script tag inclusion
- Minimal configuration required
- Default theming and behavior
- CDN-hosted for easy deployment

*[See Drop-in Integration in code-examples.md](./code-examples.md#drop-in-integration)*

### Fully Branded Integration
Complete white-label customization allows host applications to apply comprehensive branding while maintaining producer-defined structure. This approach provides maximum visual customization within functional boundaries.

**Capabilities:**
- Complete visual theme override
- Custom brand identity integration
- Component-level styling control
- Layout and spacing customization

**Limitations**: Structure and functionality remain producer-controlled to ensure consistent behavior and maintainability.

*[See Fully Branded Integration in code-examples.md](./code-examples.md#fully-branded-integration)*

### Enterprise Security Integration
Maximum isolation deployment for high-security environments with strict CSP policies, authentication requirements, and audit trails. This approach prioritizes security over convenience.

**Security Features:**
- Iframe sandbox isolation
- Custom authentication integration
- CSP-compliant resource loading
- Audit logging and monitoring

**Trade-offs**: Increased complexity in exchange for maximum security and compliance.

*[See Enterprise Security Integration in code-examples.md](./code-examples.md#enterprise-security-integration)*

## Performance Characteristics

**Bundle Sizes:**
- Core framework: 18KB gzipped (includes polyfills)
- Individual components: 3-8KB gzipped each
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

## Troubleshooting Guide

### Common Integration Issues

The troubleshooting guide addresses five major categories of integration problems with systematic debugging approaches and practical solutions.

**Issue Categories:**
1. **Component Rendering**: Missing definitions, loading failures, and broken display
2. **Styling Conflicts**: CSS isolation problems, theme application failures, and visual conflicts
3. **Navigation Conflicts**: URL routing conflicts, event handling issues, and state management problems
4. **API Integration**: CORS errors, authentication failures, and network connectivity issues
5. **Performance Problems**: Memory leaks, slow loading, and interaction lag

**Debugging Approach**: Each issue category includes symptom identification, systematic debugging steps, and proven solutions with code examples.

*[See detailed troubleshooting code in code-examples.md](./code-examples.md#troubleshooting-code-examples)*

### Environment-Specific Issues

**Browser Compatibility**: Feature detection, polyfill loading, and graceful degradation strategies for legacy browser support.

**Content Security Policy**: CSP violation detection, compliant initialization patterns, and security-first configuration approaches.

**Development vs Production**: Environment-specific debugging tools, performance monitoring, and error reporting integration.

*[See environment-specific solutions in code-examples.md](./code-examples.md#browser-compatibility-check)*

### Debugging Tools

**Development Mode**: Comprehensive logging, performance metrics, and runtime inspection tools for development environments.

**Error Reporting**: Structured error collection, monitoring service integration, and production debugging capabilities.

**Performance Analysis**: Memory usage tracking, bundle analysis, and runtime performance monitoring.

*[See debugging tools in code-examples.md](./code-examples.md#development-debugging-tools)*

### Quick Fixes Checklist

When experiencing issues, check these items in order:

1. **✅ Scripts Loaded**: Verify all Embedy scripts are loaded before component usage
2. **✅ CSP Compliance**: Check browser console for CSP violations
3. **✅ Theme Variables**: Ensure CSS custom properties follow `--embedy-*` naming
4. **✅ API Endpoints**: Verify CORS configuration and authentication
5. **✅ Browser Support**: Check for required features and load polyfills
6. **✅ Navigation**: Ensure proper isolation configuration
7. **✅ Memory**: Monitor for memory leaks in long-running applications
8. **✅ Error Handling**: Implement proper error boundaries and handlers

---

*For complete code examples and implementation details, see [code-examples.md](./code-examples.md)*