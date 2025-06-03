# Security Architecture

This document defines the comprehensive security requirements and architecture for Embedy components, covering origin validation, encryption, CSP compliance, and clickjacking protection.

## Security Principles

### Defense in Depth
Embedy employs multiple overlapping security layers to ensure robust protection even if one layer is compromised. Each security mechanism operates independently while contributing to overall system security.

### Fail-Secure Design
When security features encounter errors or ambiguous situations, the system defaults to the most restrictive behavior. Components refuse to operate rather than potentially exposing vulnerabilities.

### Adaptive Security
Security mechanisms automatically detect and adapt to the deployment environment, applying appropriate protection levels based on available features and constraints.

## Origin Validation for PostMessage Communication

### Overview
All cross-origin communication must be validated to prevent unauthorized access and data leakage. The origin validation system provides flexible configuration while maintaining strict security defaults.

### Validation Requirements

#### Multi-Layered Validation
The system must implement multiple validation strategies:
- **Explicit allowlists**: Exact origin matching for known trusted sources
- **Pattern matching**: Flexible rules for dynamic environments (e.g., development servers)
- **Subdomain validation**: Controlled trust extension to subdomains with security checks
- **Blocklist enforcement**: Permanent rejection of known malicious origins

#### Validation Process
1. Normalize all origins to a consistent format (protocol, hostname, port)
2. Check blocklists first to fail fast on known threats
3. Validate against explicit allowlists for immediate approval
4. Apply pattern matching for flexible rules
5. Perform subdomain validation with additional security checks
6. Cache validation results for performance optimization

#### Subdomain Security
When allowing subdomain trust:
- Validate proper subdomain structure (must have additional parts beyond base domain)
- Reject suspicious subdomain patterns (admin, api, secure, auth, login)
- Ensure protocol consistency between subdomain and trusted base
- Implement subdomain takeover prevention measures

#### Risk Assessment
Each validation result must include:
- Validation status (valid/invalid)
- Reason for decision (for debugging and logging)
- Risk level assessment (NONE, LOW, MEDIUM, HIGH)
- Caching eligibility for performance optimization

### Implementation Considerations
See `docs/code-examples.md#origin-validation-for-postmessage-communication` for reference implementation patterns.

## Encryption Key Management

### Overview
Sensitive data fields require client-side encryption using the Web Crypto API. The encryption system must balance security with performance and usability.

### Encryption Requirements

#### Field-Level Encryption
Support encryption for specific sensitive field types:
- Social Security Numbers (SSN)
- Credit card numbers
- Bank account information
- Custom sensitive fields as defined by implementers

#### Key Management Strategy
- Generate unique encryption keys per field type and context
- Use PBKDF2 for key derivation with minimum 100,000 iterations
- Implement session-based salt generation for key uniqueness
- Support key rotation without data loss
- Ensure keys are non-extractable from browser context

#### Encryption Process
1. Derive field-specific keys using multiple entropy sources
2. Generate unique initialization vectors (IV) for each encryption operation
3. Use AES-GCM for authenticated encryption (256-bit keys preferred)
4. Return encrypted data with metadata for decryption
5. Include timestamp for key rotation management

#### Entropy Requirements
Key derivation must incorporate multiple entropy sources:
- Session-specific salt (cryptographically random)
- Field type identifier
- Application context identifier
- Origin information
- Temporal data for uniqueness

#### Memory Security
- Clear key caches on session end or security events
- Zero out sensitive data buffers after use
- Implement secure cleanup on component destruction
- Prevent key material from entering browser history or storage

### Implementation Considerations
See `docs/code-examples.md#encryption-key-management` for reference implementation patterns.

## CSP Nonce Implementation

### Overview
Content Security Policy (CSP) compliance is mandatory for enterprise deployments. The system must support strict CSP environments while providing fallbacks for less restrictive contexts.

### CSP Requirements

#### Nonce Management
- Prefer server-provided nonces via meta tags
- Support client-generated nonces only in non-strict mode
- Validate nonce presence before creating inline content
- Apply nonces to both script and style elements

#### Script Security
- Validate all external script sources against trusted domains
- Require nonces for inline scripts in strict mode
- Support wildcard subdomain matching for CDN scenarios
- Reject untrusted sources with clear error messages

#### Style Security
- Apply nonces to inline style elements
- Convert inline styles to CSS custom properties when possible
- Use adopted stylesheets for dynamic styling
- Maintain visual consistency across CSP modes

#### Compliance Validation
The system must provide runtime CSP compliance checking:
- Detect missing nonces in strict mode
- Identify unsafe inline content
- Calculate compliance scores for monitoring
- Report violations with actionable remediation steps

#### Trusted Domain Management
- Maintain allowlist of trusted external resources
- Support wildcard patterns for CDN and subdomain scenarios
- Always trust same-origin resources
- Validate URLs before adding to trusted list

### Implementation Considerations
See `docs/code-examples.md#csp-nonce-implementation` for reference implementation patterns.

## Clickjacking Protection

### Overview
Embedy components must protect against clickjacking attacks when embedded in third-party contexts. Protection mechanisms adapt based on embedding requirements and security policies.

### Protection Requirements

#### Embedding Validation
- Detect embedding context (iframe vs direct inclusion)
- Validate parent origin against allowlist
- Check frame nesting depth (maximum 3 levels recommended)
- Implement bidirectional handshake for mutual validation

#### Visual Indicators
When embedded, components should:
- Display clear visual boundaries indicating embedded content
- Show the source origin to users
- Provide escape mechanisms (e.g., "open in new tab")
- Use fixed positioning to prevent overlay attacks

#### Parent Communication
Establish secure communication channels:
- Send capability handshake on initialization
- Include timestamp and origin verification
- Support authentication requirements from parent
- Implement timeout for handshake responses

#### Security Response
On detection of potential clickjacking:
- Log security events with full context
- Display user warnings with clear actions
- Support optional frame-breaking (use cautiously)
- Notify security monitoring systems

#### Allowed Parent Configuration
- Maintain explicit allowlist of embedding origins
- Support pattern matching for dynamic deployments
- Validate subdomain embedding with additional checks
- Provide clear documentation of embedding permissions

### Implementation Considerations
See `docs/code-examples.md#clickjacking-protection` for reference implementation patterns.

## Security Integration Strategy

### Unified Security Configuration
All security features should be configurable through a single, coherent configuration object that:
- Provides sensible defaults for common scenarios
- Allows fine-grained control for enterprise requirements
- Supports environment-specific overrides
- Validates configuration consistency

### Adaptive Security Levels
The system should automatically detect and adapt to:
- Available browser security features
- CSP policy restrictions
- Embedding context constraints
- Performance limitations

### Security Event Management
Implement comprehensive security event tracking:
- Log all security-relevant events with appropriate detail
- Support integration with SIEM systems
- Provide security dashboards for monitoring
- Enable forensic analysis capabilities

### Performance Considerations
Security features must not compromise usability:
- Cache validation results where safe
- Optimize cryptographic operations
- Minimize security check overhead
- Support progressive enhancement

### Graceful Degradation
When advanced security features are unavailable:
- Fall back to next-best security measures
- Notify administrators of reduced security
- Maintain core functionality where possible
- Document security limitations clearly

## Compliance and Auditing

### Security Compliance
The architecture supports compliance with:
- PCI DSS for payment card data
- HIPAA for healthcare information
- GDPR for privacy protection
- SOC 2 for service organizations

### Audit Trail Requirements
- Log all security decisions with justification
- Maintain tamper-evident audit logs
- Support compliance reporting
- Enable security incident investigation

### Security Testing
Regular security validation should include:
- Automated security scanning
- Penetration testing scenarios
- CSP compliance validation
- Cross-origin attack simulations

This architecture ensures Embedy components maintain enterprise-grade security while providing the flexibility needed for diverse integration scenarios. Implementers should refer to the code examples for detailed implementation patterns while adapting the security measures to their specific requirements.