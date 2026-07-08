# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Hash Utility Functions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires deterministic hashing for data integrity verification, content addressing, or cryptographic fingerprinting operations
- Node.js crypto module provides native, well-tested cryptographic primitives without external dependencies
- SHA-256 is selected as the hashing algorithm, providing 256-bit collision resistance suitable for non-password hashing use cases
- The pattern appears in core infrastructure code (src/core/hash.ts), suggesting foundational usage across the system
- Stable JSON serialization is paired with hashing to ensure consistent hash outputs for equivalent data structures

## Problem Statement

The system needs a consistent, secure, and performant approach to generating cryptographic hashes for data integrity, content addressing, or fingerprinting. Without standardization, teams may select incompatible algorithms, introduce external dependencies, or implement insecure custom hashing solutions that compromise data protection guarantees.

## Decision

1. SHOULD: Hash utility functions SHOULD be centralized in core infrastructure modules to ensure consistent implementation

## Policy Block

- SHOULD Hash utility functions SHOULD be centralized in core infrastructure modules to ensure consistent implementation

In scope:
- Data integrity verification and tamper detection
- Content-addressable storage and deduplication
- Cryptographic fingerprinting of data structures
- Cache key generation for deterministic content
- Digital signature input preparation

Out of scope:
- Password hashing and credential storage (use bcrypt, argon2, or scrypt)
- HMAC operations requiring secret keys
- Cryptographic signing operations (use appropriate signing algorithms)
- Random number generation (use crypto.randomBytes)

Exceptions:
- EXC-001: Legacy systems require MD5 or SHA-1 for compatibility with external APIs or protocols
- EXC-002: Performance-critical paths require faster non-cryptographic hashing (e.g., xxHash, MurmurHash)

## Rationale

- Evidence shows SHA-256 hashing via crypto.createHash in src/core/hash.ts with public API contracts (sha256, stableJson), indicating established infrastructure for deterministic hashing
- Node.js crypto module is part of the standard library, eliminating external dependencies and reducing supply chain risk while providing FIPS-validated implementations
- SHA-256 provides sufficient collision resistance (2^128 operations) for data integrity use cases without the overhead of SHA-512 or newer algorithms
- Centralized hash utilities in core modules enable consistent implementation, easier auditing, and simplified algorithm migration if future cryptographic advances require it

## Consequences

Positive:
- Consistent hashing behavior across the codebase reduces integration bugs and ensures interoperability
- Zero external dependencies for cryptographic hashing reduces attack surface and simplifies security audits
- Centralized implementation in core modules enables system-wide algorithm updates if cryptographic requirements change
- Stable JSON serialization paired with hashing ensures deterministic outputs for equivalent data structures

Negative:
- SHA-256 is slower than non-cryptographic hash functions, potentially impacting performance in high-throughput scenarios
- Standardization on a single algorithm may create technical debt if future use cases require different cryptographic properties
- Developers unfamiliar with cryptographic primitives may misuse SHA-256 for inappropriate use cases like password storage
- Migration to quantum-resistant hashing algorithms in the future will require coordinated system-wide changes

## Alternatives

- Use SHA-512 for all hashing operations to provide stronger collision resistance (rejected)
  Rejected because: SHA-256 provides sufficient security for data integrity use cases (2^128 collision resistance) while offering better performance and smaller hash outputs (32 bytes vs 64 bytes)
  When valid: Consider for use cases requiring 256-bit collision resistance or compliance requirements mandating SHA-512
- Use non-cryptographic hash functions (xxHash, MurmurHash) for better performance (rejected)
  Rejected because: Non-cryptographic hashes lack collision resistance and are vulnerable to intentional collision attacks, unsuitable for security-sensitive data integrity verification
  When valid: Acceptable for performance-critical internal use cases where adversarial input is not a concern (e.g., in-memory hash tables, non-security cache keys)
- Allow teams to select hashing algorithms based on specific use case requirements (rejected)
  Rejected because: Decentralized algorithm selection leads to inconsistent implementations, integration challenges, and increased security audit complexity
  When valid: May be necessary for specialized use cases with documented security review and explicit exception approval

## Risks

- Developers may misuse SHA-256 for password hashing, creating security vulnerabilities
  Mitigation: Provide clear documentation, linting rules, and code review guidelines explicitly prohibiting SHA-256 for password storage. Offer approved password hashing utilities (bcrypt, argon2).
  Owner: Security team and engineering leadership
- Performance bottlenecks may emerge in high-throughput scenarios requiring frequent hashing
  Mitigation: Profile performance-critical paths and establish exception process for non-cryptographic hashing where security properties are not required. Consider caching hash results for immutable data.
  Owner: Performance engineering team
- Future quantum computing advances may compromise SHA-256 collision resistance
  Mitigation: Monitor NIST post-quantum cryptography standards and maintain centralized hash implementation to enable system-wide algorithm migration when quantum-resistant alternatives mature.
  Owner: Security architecture team

## Implementation Notes

- Import hash utilities from src/core/hash.ts rather than directly calling crypto.createHash to ensure consistent implementation and enable future algorithm updates
- When hashing complex objects, use the stableJson utility to serialize data structures in canonical form before hashing to ensure deterministic outputs
- For password storage, use dedicated password hashing libraries (bcrypt, argon2, scrypt) with appropriate work factors—never use SHA-256 directly
- Document the purpose of each hash operation in code comments to clarify whether cryptographic properties are required for the specific use case
- Consider caching hash results for immutable data structures to avoid redundant computation in performance-sensitive paths

## Continuation Context


Verify commands:
- grep -r "createHash" --include="*.ts" --include="*.js" | grep -v "src/core/hash.ts" | grep -v "test" | grep -v "spec"
- grep -r "crypto\.createHash" --include="*.ts" --include="*.js" | grep -v "'sha256'" | grep -v '"sha256"'
- npm test -- --grep "hash.*deterministic" || echo 'Add tests for deterministic hashing behavior'

Accept when:
- All direct crypto.createHash calls outside src/core/hash.ts are migrated to use centralized hash utilities
- No SHA-256 usage is found in password storage or authentication credential handling code paths
- Test suite includes verification that equivalent data structures produce identical hash outputs via stable serialization

## Enforcement

- Verified by: Automated linting rules detect direct crypto.createHash usage outside approved core modules
- Verified by: Code review checklist includes verification that hashing use cases are appropriate for SHA-256
- Verified by: Security-focused test suite validates deterministic hashing behavior and prevents password hashing misuse
- Violation handling: CI pipeline fails on detection of direct crypto.createHash calls outside core infrastructure modules
- Violation handling: Code review blocks merge requests that use SHA-256 for password storage or authentication
- Violation handling: Security team conducts quarterly audits of cryptographic primitive usage and flags violations for remediation
- Exception process: Submit exception request to security team with documented justification, use case analysis, and risk assessment
- Exception process: Architecture review board evaluates tradeoffs for performance-critical non-cryptographic hashing exceptions
- Exception process: Approved exceptions are documented in ADR amendments with explicit scope, duration, and compensating controls