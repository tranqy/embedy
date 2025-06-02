# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Embedy is a comprehensive toolkit for creating embeddable applications with three distinct deployment methods:

1. **Web Components** (native) - Using Lit 3.x framework with shadow DOM isolation
2. **React Components** - Framework-native integration with TypeScript support  
3. **Iframe Sandboxes** - Maximum security isolation for enterprise environments

The core architecture enables universal embedding where the same application logic deploys across all three methods while maintaining consistent functionality and complete brandability.

## Architecture Principles

### Multi-Target Component System
- **Base Components**: Built with Lit 3.x as web components with TypeScript decorators
- **React Wrappers**: React components that wrap or reimplement the base components
- **Iframe Integration**: Secure communication via Postmate library for cross-origin messaging

### Theming System
- **CSS Custom Properties**: Foundation using design tokens for consistent theming
- **Runtime Theme Engine**: Dynamic theme switching without re-rendering
- **Multi-Level Customization**: Brand identity, component-level styling, and layout options
- **Dark Mode Support**: Automatic support via CSS media queries and manual controls

### Security and Isolation
- **Adaptive Security Model**: Automatic detection of environment constraints (CSP, payments, privacy)
- **Three Isolation Levels**: 
  - `IFRAME_SANDBOX` - Maximum isolation for strict CSP/payment environments
  - `SHADOW_DOM_ISOLATED` - Style and DOM isolation for privacy requirements  
  - `COMPONENT_BOUNDARY` - Standard component model for normal integration

### Navigation Architecture
- **Independent Navigation**: Embedy apps maintain their own navigation that doesn't conflict with host
- **Visual Boundaries**: Clear visual separation between embedded and host navigation elements
- **State Isolation**: Navigation state managed independently via namespace, hash routing, or postMessage
- **Conflict Prevention**: Automatic detection and mitigation of navigation conflicts with host application

### Technology Stack
- **Build System**: Vite 5.x with esbuild for development, Rollup for production
- **Validation**: Zod 3.x with React Hook Form integration
- **Documentation**: Storybook 8.x for component showcase and testing
- **PostMessage**: Postmate library for secure iframe communication (~1.6KB)

## Key Implementation Patterns

### Component Base Class
All components extend `EmbedyBase` with standardized theming and isolation properties:
```typescript
@customElement('embedy-base')
export class EmbedyBase extends LitElement {
  @property({ type: Object }) theme = {};
  @property({ type: String }) isolationLevel = 'shadow';
}
```

### Schema-Driven Configuration
Components use JSON schemas for declarative configuration with TypeScript inference:
- Form definitions with fields, validation, layout, and branding
- Atomic component library with consistent composition patterns
- Theme configurations with brand identity and component-level styling

### Progressive Enhancement
Features load based on environment capabilities:
- Core features always loaded (~15KB)
- Advanced features conditionally loaded based on browser support and bandwidth
- Mobile optimizations for touch devices
- Automatic fallbacks for legacy environments

## Development Patterns

### Bundle Strategy
- **Core Framework**: 18KB gzipped (includes polyfills)
- **Individual Components**: 3-8KB gzipped each  
- **Complete Suite**: 45KB gzipped
- **Theme Engine**: 5KB gzipped

### Performance Targets
- Core Framework: 18KB gzipped
- Time to Interactive: < 3 seconds on 3G
- Component Render: < 100ms for complex forms
- Memory Usage: < 15MB for complete form suite

### Component Development
- Use TypeScript strict mode for all components
- Implement both Lit web component and React component versions
- Include Storybook stories for documentation and testing
- Follow atomic design principles for composability
- Support all three isolation levels in component design

### Navigation Implementation
- Always namespace navigation events and routes (e.g., `embedy:navigate`, `#embedy/route`)
- Implement visual boundaries for navigation components (borders, shadows, backgrounds)
- Use CSS containment (`contain: layout style paint`) to prevent style leakage
- Support multiple navigation patterns: menu dropdowns, tabs, steppers, sidebars
- Handle navigation state via internal state, URL hash, or postMessage based on isolation level

### Security Implementation
- Always validate postMessage origins using Postmate
- Implement CSP-compliant styling (no inline styles in strict mode)
- Use sandbox attributes appropriately for iframe deployments
- Follow principle of least privilege for component permissions

### Version Control Practices
- Make atomic commits after completing each meaningful task or feature
- Write clear, descriptive commit messages that explain the "why" not just the "what"
- Follow conventional commit format when applicable (feat:, fix:, docs:, refactor:, etc.)
- Commit after each completed todo item to maintain clear project history
- Never commit broken code - ensure tests pass before committing
- Create descriptive commits that provide context for future developers
- Include the impact and reasoning behind changes in commit messages
- Avoid generic messages like "Update files" or "Fix bug" without context

## Producer vs Consumer Separation

### Producer Controls (App Developers)
- Component structure and composition
- Business logic and validation rules
- Navigation flows and routing
- Data management and API integration
- Headers, footers, and structural elements

### Consumer Controls (Host Applications)
- Visual theming and branding
- Colors, typography, spacing
- Component appearance (not structure)
- Dark mode configuration
- Localization and text content

## Integration Examples

The system supports three integration patterns:
1. **Drop-in**: Minimal setup with default branding via CDN
2. **Fully Branded**: Complete white-label customization (visual theming only, not structural changes)
3. **Enterprise Security**: Maximum isolation with custom authentication and CSP compliance

## Documentation Structure

### Core Documentation Files
- **docs/overview.md**: High-level architectural vision and system capabilities
- **docs/technical-requirements.md**: Flexible requirements focusing on "what" and "why" rather than "how"
- **docs/code-examples.md**: Concrete implementation examples for reference
- **docs/multi-target-architecture.md**: Web components and React integration patterns
- **docs/build-system-architecture.md**: Progressive loading and bundle optimization details
- **docs/security-architecture.md**: Comprehensive security implementation guidance
- **docs/unanswered-questions.md**: Critical architectural decisions requiring team input

### Documentation Philosophy
The documentation intentionally avoids prescriptive implementation details to give implementers freedom within architectural constraints. Code examples are separated from requirements to serve as reference rather than mandatory patterns.

## Key Architectural Decisions

### Progressive Loading Strategy
- Core bundle must remain under 18KB gzipped
- Features load based on device capabilities (bandwidth, memory, browser support)
- Polyfills load conditionally to avoid penalizing modern browsers
- Bundle splitting by feature enables optimal caching

### Security Model
- Three isolation levels adapt to environment constraints automatically
- CSP compliance is mandatory for payment and high-security environments
- All cross-origin communication uses Postmate for security validation
- Privacy features include client-side encryption for sensitive fields

### Performance Requirements
- Time to Interactive: < 3 seconds on 3G networks
- Component render: < 100ms for complex forms
- Memory usage: < 15MB for complete suite
- Automatic performance regression detection in CI/CD

## Areas Requiring Decisions

### Critical Open Questions
1. **JSON Schema vs TypeScript**: Which source of truth for form definitions?
2. **State Management**: Component-local or centralized store pattern?
3. **Build Output**: Single bundle with tree-shaking or pre-split packages?
4. **Theme Inheritance**: How do component themes inherit from global themes?
5. **Testing Strategy**: Unit tests only or include visual regression testing?

### Future Considerations
- WebAssembly integration for compute-intensive validations
- Service Worker support for offline functionality
- Micro-frontend architecture compatibility
- Server-side rendering support for SEO requirements

## Development Workflow Recommendations

### Component Development Process
1. Start with Lit web component implementation
2. Generate React wrapper automatically via build tools
3. Test all three deployment methods (native, React, iframe)
4. Validate security isolation at each level
5. Performance test with throttled network conditions

### Testing Approach
- Unit tests for all business logic
- Integration tests for cross-component communication
- Security tests for isolation boundaries
- Performance benchmarks for bundle size and runtime
- Accessibility testing with automated tools

## Implementation Priorities

### Phase 1 Focus Areas
- Core form components (input, select, textarea)
- Basic theming system with CSS custom properties
- Security adapter for environment detection
- Build system with progressive loading

### Technical Debt Prevention
- Maintain strict TypeScript types from day one
- Document architectural decisions in ADRs
- Regular bundle size audits
- Security review for each new component
- Performance profiling in CI pipeline