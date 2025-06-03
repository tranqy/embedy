# Multi-Target Architecture

This document outlines the conceptual architecture that enables Embedy components to seamlessly deploy across web components, React components, and iframe sandboxes while maintaining consistent functionality and developer experience.

## Core Philosophy

The multi-target architecture is built on the principle of "write once, deploy everywhere" - components are authored once using modern web standards and automatically adapted to work in any JavaScript environment. This approach eliminates the need for manual porting between frameworks while ensuring optimal integration patterns for each deployment method.

## Component Interface Foundation

At the heart of the multi-target system is a unified component interface that defines the contract all Embedy components must fulfill. This interface ensures that regardless of how a component is deployed, it will expose the same core functionality, properties, and events.

The interface establishes:
- **Core Methods**: Standard operations like validate, submit, reset, and updateTheme that work identically across all deployment methods
- **Universal Properties**: Configuration options like theme, isolation level, and disabled state that behave consistently
- **Event Contract**: A standardized event system that translates seamlessly between custom events, React callbacks, and postMessage communication

This shared interface acts as the foundation for feature parity across deployment methods, ensuring developers can switch between web components, React components, and iframe deployments without changing their integration code.

## Automatic React Wrapper Generation

One of the key innovations in the multi-target architecture is the automatic generation of React wrappers from web components. This system eliminates the traditional burden of maintaining separate React and web component implementations.

### Build-Time Code Generation

The React wrapper generation happens at build time through an intelligent code generation system that:

1. **Analyzes Web Components**: Extracts property definitions, event signatures, and type information from Lit components
2. **Generates React Components**: Creates idiomatic React components that properly handle props, refs, and lifecycle
3. **Preserves Type Safety**: Maintains full TypeScript support with proper type inference and checking
4. **Handles Edge Cases**: Manages ref forwarding, event delegation, and property synchronization automatically

### Type-Safe Property System

The architecture includes a sophisticated property decoration system that captures type information at design time. This metadata is used during the build process to generate React components with proper TypeScript interfaces, ensuring type safety across framework boundaries.

Key benefits of this approach:
- **Zero Runtime Overhead**: All wrapper generation happens at build time
- **Framework-Native Patterns**: Generated React components follow React best practices
- **Automatic Updates**: Changes to web components are automatically reflected in React wrappers
- **Type Inference**: TypeScript types flow seamlessly from web components to React

## Feature Parity Management

Ensuring consistent functionality across different deployment methods requires sophisticated capability detection and feature adaptation. The architecture includes a comprehensive system for managing feature parity while respecting the unique capabilities of each deployment environment.

### Capability Detection System

The capability detection system automatically identifies what features are available in each deployment context:

**Web Component Capabilities**:
- Direct DOM access for optimal performance
- Shadow DOM for style isolation
- Custom events for native browser integration
- CSS custom properties for theming

**React Component Capabilities**:
- React hooks for state management
- Context API for theme propagation
- JSX props for configuration
- Virtual DOM for efficient updates

**Iframe Sandbox Capabilities**:
- PostMessage for secure communication
- Complete isolation from host page
- Cross-origin deployment support
- Independent security context

### Adaptive Feature Implementation

The architecture intelligently selects the appropriate implementation for each feature based on the deployment context. For example:
- **Theme Management**: Uses CSS custom properties in web components, React Context in React apps, and postMessage in iframes
- **State Management**: Leverages reactive properties in web components, hooks in React, and message passing in iframes
- **Event Handling**: Employs custom events for web components, callbacks for React, and postMessage for iframes

This adaptive approach ensures optimal performance and natural integration patterns for each deployment method while maintaining functional equivalence.

## React Adaptation Layer

The React adaptation layer bridges the gap between web component patterns and React idioms, ensuring that components feel native to React developers while maintaining the benefits of the underlying web component architecture.

### Key Adaptation Strategies

**Shadow DOM in React**: While React doesn't natively support shadow DOM, the adaptation layer provides shadow DOM encapsulation for React components when needed. This is particularly valuable for maintaining style isolation in complex applications.

**Event System Translation**: The layer automatically translates between web component custom events and React's callback prop pattern. This ensures that React developers can use familiar patterns like `onChange` and `onSubmit` while the underlying web component uses custom events.

**Property Synchronization**: Reactive properties from Lit components are seamlessly synchronized with React state, ensuring that updates flow properly through React's rendering pipeline while maintaining the web component's internal state management.

**Lifecycle Coordination**: The adaptation layer manages the coordination between React lifecycle methods and web component lifecycle callbacks, ensuring proper initialization, updates, and cleanup across framework boundaries.

### Benefits of the Adaptation Approach

1. **Framework-Native Experience**: React developers work with components that feel like native React components
2. **Preserved Web Standards**: The underlying web components remain standards-compliant and framework-agnostic
3. **Performance Optimization**: The adaptation layer is optimized to minimize overhead and prevent unnecessary re-renders
4. **Progressive Enhancement**: Features are progressively enhanced based on React capabilities while maintaining baseline functionality

## Type System Integration

A unified type system ensures type safety across all deployment methods, providing developers with consistent interfaces and excellent IDE support regardless of how components are deployed.

### Shared Type Architecture

The type system is built on several key principles:

1. **Single Source of Truth**: Component types are defined once and automatically propagated to all deployment methods
2. **Framework-Specific Extensions**: Each deployment method can extend base types with framework-specific properties
3. **Type Inference**: The system automatically infers types from component implementations
4. **Build-Time Validation**: Type checking happens at build time to catch integration errors early

### Type Flow Across Boundaries

Types flow seamlessly from web components to React components through the build process:
- Property decorators capture type information from web components
- Build tools generate TypeScript interfaces for React components
- Runtime validation ensures type safety in production
- IDE support provides autocomplete and type checking across frameworks

## Implementation Benefits

The multi-target architecture delivers significant benefits:

### For Developers
1. **Write Once**: Author components once using modern web standards
2. **Deploy Everywhere**: Components work in any JavaScript environment
3. **Type Safety**: Full TypeScript support across all deployment methods
4. **Natural Integration**: Components follow the idioms of each framework
5. **Progressive Enhancement**: Features adapt to available capabilities

### For Applications
1. **Performance**: Optimal implementation for each deployment context
2. **Compatibility**: Works with legacy systems and modern frameworks
3. **Security**: Appropriate isolation levels for different requirements
4. **Maintainability**: Single codebase reduces maintenance burden
5. **Future-Proof**: Based on web standards, not framework-specific APIs

### For End Users
1. **Consistent Experience**: Same functionality regardless of integration method
2. **Better Performance**: Optimized for each deployment context
3. **Enhanced Security**: Appropriate isolation based on requirements
4. **Accessibility**: Standards-based approach ensures accessibility
5. **Reliability**: Reduced complexity means fewer bugs

## Architecture Evolution

The multi-target architecture is designed to evolve with the web platform:

- **New Frameworks**: The pattern can be extended to support new frameworks
- **Web Standards**: As new standards emerge, they can be incorporated
- **Performance Improvements**: The architecture can adopt new optimization techniques
- **Security Enhancements**: New isolation strategies can be added as needed

This forward-looking design ensures that Embedy components will continue to work well as the web platform evolves, protecting the investment in component development while enabling adoption of new technologies as they mature.