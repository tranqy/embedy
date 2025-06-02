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

**Multi-Target Architecture**:
The framework implements a unified component interface that works across all deployment methods. Each component extends a standardized base class that provides common functionality for theming, validation, and lifecycle management. The architecture maintains clear separation between producer-controlled structure and consumer-controlled theming.

**Component Base Architecture**: All components inherit from a base class that defines the standard interface including validation methods, theme management, and isolation controls. The base class uses TypeScript decorators for property definition and provides abstract methods for producers to implement their specific business logic.

**React Wrapper Generation**: React components are automatically generated at build time using build-time reflection. The system creates wrapper components that handle prop synchronization, event adaptation, and React-specific patterns like refs and lifecycle hooks. This ensures React developers get a native React experience while maintaining the underlying web component architecture.

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

**Progressive Loading Architecture**:
The build system implements intelligent progressive loading that adapts to device capabilities and network conditions.

**Capability Detection**: The system analyzes device characteristics including network speed, available memory, touch support, screen dimensions, and browser feature availability to determine optimal loading strategies.

**Bundle Splitting Strategy**: Uses a three-tier approach with a core bundle containing essential functionality, feature-specific chunks loaded based on capabilities, and optimally cached vendor dependencies. The system prioritizes critical path loading while deferring advanced features for capable devices.

**Polyfill Management**: Implements conditional polyfill loading based on feature detection, covering web components APIs, modern JavaScript features, and browser-specific capabilities. The system maintains compatibility across legacy and modern browsers without unnecessary overhead.

**Fallback Strategy**: Provides multi-tier fallback handling including retry logic with exponential backoff, alternative CDN sources, local vendor bundles, and graceful degradation for minimal functionality in constrained environments.

**Build Configuration**: The Vite configuration implements sophisticated chunking strategies that separate vendor libraries, feature modules, and individual components. This enables optimal caching and loading patterns while maintaining clear separation of concerns.

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

**Implementation**: Forms use declarative schema definitions with nested object validation, array validation with minimum requirements, and comprehensive error messaging. The system integrates with React Hook Form using resolver patterns that provide real-time validation feedback and seamless error state management.

### Theming and Design System

#### CSS Custom Properties + Design Tokens
**Rationale**: CSS custom properties provide the most flexible and performant theming solution while maintaining clear producer/consumer boundaries.

**Architecture**: The theming system uses CSS custom properties as the foundation, implementing a comprehensive design token system that maintains clear boundaries between consumer and producer control. The system includes brand tokens for colors and identity, semantic tokens that map to specific UI elements, and a structured spacing and typography scale. Producer-defined structural tokens control layout and component hierarchy but are not exposed to consumers. Dark mode support is implemented through CSS media queries with automatic token switching.

#### Runtime Theme Engine
The theme management system maintains strict boundaries between consumer and producer control. Consumer configuration includes brand identity elements, component visual styling, layout preferences for spacing and direction, and dark mode settings. Producer configuration controls structural elements like component inclusion, application flows, headers and navigation, business validation rules, and API integrations.

The ThemeEngine class provides methods for consumers to update visual properties by modifying CSS custom properties dynamically. Theme changes trigger events for reactive updates while maintaining isolation of structural controls that remain under producer management.

### Navigation Isolation and Consistency

#### Navigation Architecture
**Core Principle**: Embedy applications maintain independent navigation that is visually distinct and functionally isolated from the host application.

The navigation isolation strategy defines producer-controlled structure including navigation type, position, and behavior, along with isolation mechanisms for visual boundaries, state management, and conflict resolution. Consumer theming controls visual aspects like colors, borders, spacing, and typography without affecting structural elements.

The NavigationManager class implements conflict prevention by namespacing all navigation events and maintaining independent state. It supports multiple state management strategies including internal state, URL hash routing, and postMessage communication. Navigation operations are scoped to the embedy context to prevent interference with host application routing.

#### Visual Boundary Requirements

Visual boundary requirements include producer-defined positioning and z-index management, consumer-themeable borders and backgrounds with fallback values, and containment properties to prevent style leakage. Style isolation is achieved through CSS containment and style resets that prevent inheritance from host applications while maintaining component functionality.

#### Navigation Patterns

Navigation patterns are designed with varying levels of host conflict consideration. Embedded menus use relative positioning with minimal conflict potential, tab navigation may compete visually with host tabs, sidebar navigation requires dedicated space and has higher conflict potential, and stepper navigation is self-contained with minimal host interference. Each pattern defines appropriate positioning, display methods, and conflict mitigation strategies.

### Security and Isolation

#### Iframe Sandboxing
**Implementation based on modern security best practices**:

The SecurityAdapter implements environment-based isolation level detection. It maintains predefined configurations for each security level including iframe sandbox permissions, shadow DOM settings, and component boundary isolation. The system automatically determines appropriate isolation based on Content Security Policy constraints, payment handling requirements, data privacy needs, and third-party embedding context.

#### Comprehensive Security Implementation

**Security Threat Model**: Embedy components must protect against XSS, clickjacking, data exfiltration, and privacy violations while maintaining functionality across isolation levels.

##### Content Security Policy (CSP) Compliance
The system supports both standard and strict CSP implementations. Standard CSP allows necessary sources for embedded content including CDN resources, API endpoints, and trusted frame ancestors. Strict CSP for high-security environments uses nonce-based script and style loading with minimal allowed sources and restricted frame embedding.

##### Clickjacking Protection
Clickjacking protection implements embedding context validation by checking frame hierarchy and validating parent origins against allowlists. The system sets appropriate frame options headers and provides runtime validation to prevent unauthorized embedding. Protection includes both client-side detection and server-side header configuration for comprehensive security coverage.

##### Data Privacy and GDPR Compliance
Privacy management includes configurable data processing basis, retention policies, anonymization options, cross-border transfer controls, and encryption requirements. The PrivacyManager validates data handling by identifying sensitive fields, applying required encryption, and adding privacy metadata including consent timestamps and retention policies. Client-side encryption uses the Web Crypto API for sensitive data protection before transmission.

#### PostMessage Security

**Library Recommendation**: **Postmate** by Dollar Shave Club
**Rationale**: Promise-based API with built-in security validation, minimal bundle size (~1.6KB gzipped), and excellent documentation.

**Multi-Layered Security Architecture**:

**Origin Validation**: Multi-tier validation with allowlists, pattern matching, subdomain validation, and runtime verification. Supports strict mode for high-security environments and maintains blocked origin lists.

**Encryption Key Management**: Uses Web Crypto API with PBKDF2 key derivation, unique IVs per encryption, session-based salt generation, and secure key rotation. Keys are non-extractable and cached per field type and context.

**CSP Compliance**: Server-provided nonce support, strict mode detection, dynamic script/style creation with nonce application, and CSS custom property fallbacks for inline style restrictions.

**Clickjacking Protection**: Frame ancestor validation, visual embedding indicators, depth validation, parent communication handshakes, and graceful handling of legitimate embedding scenarios.

**Enhanced Security Implementation**: The SecurePostMessage class implements comprehensive message validation with allowlist-based origin checking, schema validation, and malicious content scanning. It extends Postmate with additional security layers including message encryption, integrity checking, and security header management. The MessageValidator performs structure validation, size limits, frequency checking, and XSS pattern detection to prevent malicious content transmission.

##### Cross-Origin Resource Sharing (CORS) Configuration
CORS policies are configured differently for production and development environments. Production configuration uses strict origin allowlists, limited HTTP methods, specific headers, and appropriate cache settings. Development configuration allows broader access for testing while maintaining credential security. The CORSValidator ensures proper origin validation and header verification for all cross-origin requests.

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

**Postmate Implementation**: Parent-child iframe communication uses Postmate's promise-based API with container targeting, URL specification, and custom styling. The parent listens for events from the child iframe and can call exposed methods for theme updates. The child iframe implementation creates a model with exposed methods for parent communication and can emit events back to the parent for data transmission.

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
Dynamic imports enable progressive enhancement based on device capabilities. The FeatureLoader analyzes browser capabilities, bandwidth, and device type to determine optimal feature loading. Core features are always loaded while advanced features like validation, mobile optimizations, and virtual scrolling are conditionally imported based on capability detection. The system assembles the final component library from core and available features.

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
Storybook documentation uses the web-components-vite framework with comprehensive story discovery across multiple file types. Essential addons provide core functionality, accessibility testing ensures compliance, and design token integration maintains visual consistency. Automatic documentation generation creates comprehensive component reference materials.

#### Type Definitions
Comprehensive TypeScript definitions maintain strict producer/consumer separation. The EmbedyComponent interface provides consumer-controlled theming and isolation properties alongside producer-defined callback structure. FormConfig separates producer-controlled structure including fields, layout, and validation from consumer-controlled branding. ComponentDefinition clearly delineates producer aspects covering functionality and structure from consumer aspects covering visual presentation and user experience.

## Complete API Reference

### Core Component API

#### EmbedyBase Component
The EmbedyBase class serves as the foundation for all Embedy components, providing standardized configuration properties, event callbacks, and lifecycle management. It includes theme configuration, isolation level control, API endpoint management, and comprehensive error handling. Lifecycle methods handle component initialization and cleanup with proper error boundaries, while public API methods support theme updates, validation, and form submission with detailed error reporting and event dispatching.

#### Error Types and Handling
The comprehensive error system includes the EmbedyError class with typed error categories, detailed error information, and JSON serialization capabilities. Error types cover initialization, validation, submission, theming, network, security, and configuration issues with appropriate default messages. The EmbedyErrorBoundary provides React component error handling with automatic error conversion, monitoring service integration, fallback UI rendering, and user-provided error handler support.

#### Network and API Layer
The EmbedyApiClient provides robust form submission with configurable retry logic, authentication handling, and comprehensive error management. It includes automatic retry with exponential backoff for transient errors while avoiding retries for permanent client errors. The client supports configurable base URLs, authentication tokens, retry policies, and proper error classification for different HTTP status codes.

### Integration Examples with Error Handling

#### React Integration
React integration utilizes the EmbedyProvider for configuration management, EmbedyErrorBoundary for error handling, and form components with comprehensive state management. The implementation includes loading states, error handling with monitoring service integration, theme configuration via environment variables, validation rule configuration, and error display with retry functionality. Components support standard React patterns including hooks, error boundaries, and provider context.

#### Web Components Integration
Web components integration uses vanilla JavaScript with DOM event listeners for component interaction. The implementation includes error handling with toast notifications and error tracking service integration, form submission with fetch API and proper error handling, event listener configuration for validation and error events, and utility functions for user feedback. Components are configured via HTML attributes for API endpoints, theming, and isolation levels.

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