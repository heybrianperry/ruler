# Use Node.js crypto Module with SHA-256 for Cryptographic Hashing: Hash Operations Requiring Cryptographic Strength Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires cryptographic hashing capabilities for data integrity, content addressing, or security-sensitive operations within core infrastructure
- Node.js provides a built-in crypto module that exposes standard cryptographic primitives without external dependencies
- SHA-256 is observed in use within the core hash utility module, indicating a choice of hash algorithm strength and standardization
- The pattern is localized to a single core utility file, suggesting centralized hash operation handling rather than distributed implementation

## Problem Statement

The system needs a standardized approach to cryptographic hashing that provides sufficient security strength for data protection use cases while minimizing external dependencies and maintaining consistency across hash operations.

## Decision

1. MUST: Hash operations requiring cryptographic strength MUST use SHA-256 as the hash algorithm

## Policy Block

- MUST Hash operations requiring cryptographic strength MUST use SHA-256 as the hash algorithm

In scope:
- Core utility modules providing hash operations
- Security-sensitive data processing requiring cryptographic-strength hashing
- Content addressing or data integrity verification use cases
- Operations requiring non-reversible data transformation

Out of scope:
- Non-cryptographic hash operations where performance is critical and security is not required (use faster non-crypto hashes)
- Password hashing operations (use dedicated password hashing algorithms with salt and iteration)
- Client-side browser environments where Node.js crypto module is unavailable

## Rationale

- The Node.js crypto module provides battle-tested cryptographic primitives maintained by the Node.js security team, eliminating the need for external dependencies and their associated supply chain risks
- SHA-256 offers sufficient cryptographic strength (256-bit output) for most data protection use cases while being widely supported and standardized
- Centralizing hash operations in core utilities promotes consistency and makes algorithm changes or security updates easier to implement across the codebase
- The evidence shows this pattern established in core infrastructure with high significance (0.90), indicating deliberate architectural placement

## Consequences

Positive:
- Zero external dependencies for cryptographic hashing reduces supply chain attack surface and dependency maintenance burden
- SHA-256 provides strong collision resistance and preimage resistance suitable for security-sensitive operations
- Centralized implementation in core utilities enables consistent hash behavior and simplified security auditing
- Built-in module ensures compatibility across Node.js versions and environments without installation overhead

Negative:
- Node.js crypto module ties the implementation to Node.js runtime, limiting portability to browser or other JavaScript environments
- SHA-256 is slower than non-cryptographic hash functions for use cases that do not require cryptographic strength
- Single-file evidence suggests limited adoption across the codebase, potentially indicating incomplete pattern implementation
- Built-in crypto module API is lower-level than some third-party libraries, requiring more careful implementation of security-sensitive operations

## Alternatives

- Use third-party cryptographic libraries with higher-level APIs and additional features (rejected)
  Rejected because: Evidence shows adoption of built-in crypto module, avoiding external dependencies and their associated maintenance and security risks
  When valid: When advanced cryptographic features beyond basic hashing are required, or when higher-level abstractions would significantly improve security posture
- Use faster non-cryptographic hash functions for all hashing operations (rejected)
  Rejected because: The security.encryption facet and Data Protection category indicate security-sensitive requirements that demand cryptographic-strength hashing
  When valid: For performance-critical operations where data integrity is needed but cryptographic security is not required
- Use stronger hash algorithms like SHA-512 or SHA-3 (deferred)
  Rejected because: Evidence specifically shows SHA-256 in use; stronger algorithms may be considered if threat model requires additional security margin
  When valid: When threat analysis indicates SHA-256 collision resistance or output length is insufficient for the use case

## Risks

- Incorrect usage of crypto module APIs could introduce security vulnerabilities despite using strong algorithms
  Mitigation: Centralize hash operations in well-tested core utilities; conduct security review of crypto module usage; implement comprehensive test coverage
  Owner: engineering team
- Limited evidence (single file) suggests pattern may not be consistently applied across the codebase
  Mitigation: Audit codebase for other hash operations; migrate inconsistent implementations to centralized utilities; document pattern in developer guidelines
  Owner: engineering team
- SHA-256 alone is insufficient for password hashing and may be misapplied to authentication use cases
  Mitigation: Clearly document scope limitations; use dedicated password hashing algorithms (bcrypt, argon2) for authentication; enforce through code review
  Owner: engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- The crypto module is built into Node.js; verify the Node.js version in use supports the required crypto APIs by consulting the project's runtime version specification
- When implementing hash operations, always specify the algorithm explicitly in createHash calls rather than relying on defaults
- For data integrity use cases, consider whether HMAC (keyed hashing) provides better security properties than plain hashing for your specific threat model

## Continuation Context


Verify commands:
- Discover the project's static analysis or linting configuration and execute the security-focused rules to verify crypto module usage patterns
- Locate the project's test suite and execute tests covering the core hash utility module to verify SHA-256 implementation correctness
- Search the codebase for all hash-related operations and verify they route through centralized utilities rather than inline crypto module usage

Accept when:
- All cryptographic hash operations use the crypto module's createHash API with SHA-256 algorithm specification
- No usage of deprecated or weak hash algorithms (MD5, SHA-1) exists in security-sensitive code paths
- Hash operations are centralized in core utility modules with test coverage demonstrating correct implementation

## Enforcement

- Verified by: Code review process checking for direct crypto module usage outside approved utility modules
- Verified by: Static analysis tools scanning for weak hash algorithm usage
- Verified by: Security audit reviewing cryptographic primitive usage patterns
- Violation handling: Violations identified in code review must be refactored to use centralized hash utilities before merge
- Violation handling: Usage of weak hash algorithms in security-sensitive contexts triggers security review and mandatory remediation
- Violation handling: Direct crypto module usage outside core utilities requires architectural review and justification
- Exception process: Exception requests must document the specific use case and why centralized utilities are insufficient
- Exception process: Security team must review and approve exceptions involving cryptographic operations
- Exception process: Approved exceptions must be documented with rationale and reviewed annually