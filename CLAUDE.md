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