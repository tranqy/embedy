# Unanswered Questions and Documentation Gaps

This document tracks questions and gaps identified in the current Embedy documentation that need to be addressed to provide complete implementation guidance.

## Architecture & Design Questions

### Multi-Target Component System
- ✅ **ANSWERED**: How does the automatic React wrapper generation work from Lit components?
- ✅ **ANSWERED**: What is the exact mechanism for maintaining feature parity across all three deployment methods?
- ✅ **ANSWERED**: How are TypeScript types shared between web components and React wrappers?
- ✅ **ANSWERED**: What happens when a Lit component feature can't be directly translated to React patterns?

### Environment Detection & Adaptation
- What specific CSP directives trigger iframe sandbox mode vs shadow DOM isolation?
- How does the framework detection work in practice? Does it check for global variables, import patterns, or bundler configurations?
- What bandwidth thresholds determine feature loading strategies?
- How are navigation conflicts detected automatically? What specific patterns are scanned?

### Security Implementation Details
- ✅ **ANSWERED**: What is the exact origin validation process for PostMessage communication?
- ✅ **ANSWERED**: How are encryption keys managed for sensitive data fields?
- ✅ **ANSWERED**: What specific CSP nonce implementation is required for strict CSP environments?
- ✅ **ANSWERED**: How does the clickjacking protection work with legitimate iframe embedding?

## Technical Implementation Questions

### Build System & Bundle Management
- ✅ **ANSWERED**: How does the progressive feature loading determine which features to load?
- ✅ **ANSWERED**: What is the exact bundle splitting strategy? How are dependencies shared between chunks?
- ✅ **ANSWERED**: How are polyfills conditionally loaded based on browser detection?
- ✅ **ANSWERED**: What is the fallback strategy when dynamic imports fail?

### Performance Optimization
- How is the 18KB core bundle achieved? What specific optimizations are used?
- What are the actual memory usage patterns for the component lifecycle?
- How is virtual scrolling implemented for large datasets?
- What specific debouncing strategies are used for validation and API calls?

### Theming Engine Deep Dive
- How does runtime theme switching work without re-rendering? What's the exact mechanism?
- How are CSS custom property cascading conflicts resolved?
- What happens when host application styles conflict with Embedy component styles?
- How is theme validation implemented to prevent invalid configurations?

## Component Library Questions

### Schema-Driven Configuration
- How are JSON schemas converted to TypeScript types at runtime?
- What is the validation pipeline for form configurations?
- How do schema versions and migrations work for evolving form definitions?
- What happens when schema validation fails at runtime?

### Form Components Implementation
- How does the client lookup component handle large datasets and search performance?
- What is the exact currency formatting implementation for international support?
- How does the tax calculator handle different regional tax rules?
- What validation strategies are used for complex business rules?

### Navigation System
- How does the namespace collision detection work in practice?
- What is the exact implementation of URL hash routing with namespace prefixing?
- How are navigation events prevented from bubbling to host applications?
- What happens when multiple Embedy instances exist on the same page?

## Integration & Deployment Questions

### iframe Sandbox Implementation
- What is the exact postMessage protocol used for communication?
- How are authentication tokens securely passed to iframe content?
- What sandbox attributes are required for different security levels?
- How does theme injection work across iframe boundaries?

### React Integration Details
- How does the React Hook Form integration handle custom validation rules?
- What is the exact Provider pattern implementation for configuration?
- How are React error boundaries integrated with Embedy error handling?
- How does the component lifecycle align with React's rendering cycle?

### Web Components Integration
- How does shadow DOM isolation handle global CSS frameworks like Bootstrap?
- What is the custom element registration strategy to avoid naming conflicts?
- How are event listeners cleaned up when components are unmounted?
- What happens when web components are used in SSR environments?

## Development & Debugging Questions

### Error Handling & Monitoring
- What is the complete error taxonomy and error code system?
- How are errors reported to external monitoring services?
- What debugging tools are available in development mode?
- How do error boundaries work across different integration methods?

### Testing Strategy
- What testing approach is used for cross-browser compatibility?
- How are the three different embedding methods tested in parallel?
- What visual regression testing is implemented for theming?
- How is security testing automated for iframe implementations?

### Documentation & Developer Experience
- How is API documentation auto-generated from TypeScript definitions?
- What interactive examples are available in Storybook?
- How do developers test theme configurations during development?
- What migration guides exist for version upgrades?

## Business Logic & Use Cases

### Form Validation & Processing
- How are complex validation rules (like tax calculations) implemented?
- What is the data flow for multi-step form wizards?
- How are partial saves and draft management handled?
- What accessibility features are built into form components?

### Enterprise Features
- How does audit logging work across different isolation levels?
- What compliance features (GDPR, PCI) are built into the framework?
- How are user permissions and role-based access controls implemented?
- What enterprise authentication patterns are supported?

### Internationalization & Localization
- How are translations loaded and cached for different locales?
- What right-to-left (RTL) language support is provided?
- How are date, number, and currency formatting handled regionally?
- What font loading strategies are used for different character sets?

## Performance & Scalability Questions

### Resource Management
- What caching strategies are used for component definitions and themes?
- How does component cleanup prevent memory leaks in long-running applications?
- What are the performance implications of having multiple embedded forms?
- How does the system handle high-frequency form submissions?

### Network & API Integration
- What retry mechanisms are implemented for network failures?
- How are rate limits handled when submitting forms?
- What offline capabilities are available for form completion?
- How does the system handle large file uploads in embedded contexts?

## Browser Support & Compatibility

### Legacy Browser Support
- What specific polyfills are required for IE11 support?
- How does graceful degradation work when modern features aren't available?
- What are the performance trade-offs for legacy browser support?
- How are touch interactions optimized for mobile browsers?

### Modern Browser Features
- How does the system leverage modern web APIs like Web Workers?
- What progressive web app features are available in embedded contexts?
- How are newer CSS features like container queries utilized?
- What service worker integration is possible for embedded components?

## Missing Implementation Examples

### Real-World Integration Scenarios
- Complete e-commerce checkout form embedded in shopping cart
- Multi-tenant SaaS application with white-label form embedding
- Government form system with strict security requirements
- Mobile app integration using WebView components

### Advanced Customization Examples
- Custom validation rule implementation
- Complex theme inheritance patterns
- Dynamic form generation from API responses
- Integration with existing design systems

### Troubleshooting Scenarios
- Debugging CSP violations in production
- Resolving theme conflicts with host application CSS
- Performance optimization for large form datasets
- Security audit compliance verification

---

## Priority Assessment

### High Priority (Implementation Blockers)
- Build system configuration and bundle management
- Security implementation details (CSP, iframe sandbox)
- Theme engine runtime behavior
- Cross-framework type safety

### Medium Priority (Developer Experience)
- Debugging tools and error handling
- Performance optimization strategies
- Testing approaches
- Documentation generation

### Low Priority (Advanced Features)
- Enterprise compliance features
- Advanced customization patterns
- Edge case handling
- Legacy browser optimizations

---

*This document should be regularly updated as questions are answered and new gaps are identified during implementation.*