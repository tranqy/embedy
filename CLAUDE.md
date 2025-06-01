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
- **Core Framework**: ~15KB gzipped
- **Individual Components**: 3-8KB each  
- **Complete Suite**: ~45KB gzipped
- **Theme Engine**: ~5KB gzipped

### Performance Targets
- Initial Load: < 50KB gzipped
- Time to Interactive: < 2 seconds on 3G
- Component Render: < 16ms (60fps)
- Memory Usage: < 10MB for complete form suite

### Component Development
- Use TypeScript strict mode for all components
- Implement both Lit web component and React component versions
- Include Storybook stories for documentation and testing
- Follow atomic design principles for composability
- Support all three isolation levels in component design

### Security Implementation
- Always validate postMessage origins using Postmate
- Implement CSP-compliant styling (no inline styles in strict mode)
- Use sandbox attributes appropriately for iframe deployments
- Follow principle of least privilege for component permissions

## Integration Examples

The system supports three integration patterns:
1. **Drop-in**: Minimal setup with default branding via CDN
2. **Fully Branded**: Complete white-label customization with custom headers/footers  
3. **Enterprise Security**: Maximum isolation with custom authentication and CSP compliance