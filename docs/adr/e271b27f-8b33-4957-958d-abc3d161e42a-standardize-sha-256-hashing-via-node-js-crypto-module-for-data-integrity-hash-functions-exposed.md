# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Hash Functions Exposed

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires deterministic content hashing for data integrity verification and content-addressable storage patterns
- SHA-256 provides cryptographic-strength hashing suitable for detecting data tampering and ensuring content immutability
- The Node.js crypto module is available as a built-in library, eliminating external dependencies for core hashing operations
- Stable JSON serialization is required before hashing to ensure consistent hash outputs for equivalent data structures
- The pattern is observed in core infrastructure code (src/core/hash.ts) suggesting foundational architectural significance

## Problem Statement

The system needs a standardized, cryptographically secure method for generating content hashes that can reliably identify data integrity violations, support content-addressable storage, and provide deterministic outputs across different execution contexts without introducing external dependencies.

## Decision

1. MUST: Hash functions MUST be exposed through public API contracts (sha256, stableJson) for consistent usage across the codebase

## Policy Block

- MUST Hash functions MUST be exposed through public API contracts (sha256, stableJson) for consistent usage across the codebase

In scope:
- Data integrity verification operations
- Content-addressable storage key generation
- Deterministic content fingerprinting
- Cryptographic hash generation for non-password use cases
- Core infrastructure hashing utilities

Out of scope:
- Password hashing (use bcrypt, argon2, or similar)
- HMAC operations requiring secret keys
- Digital signatures requiring asymmetric cryptography
- Hash functions for hash tables or non-cryptographic purposes
- Legacy systems with established hashing mechanisms

Exceptions:
- EXC-001: Integration with third-party systems that mandate specific hashing algorithms other than SHA-256
- EXC-002: Performance-critical paths where SHA-256 overhead is demonstrably prohibitive and weaker hashing is acceptable

## Rationale

- The Node.js crypto module provides battle-tested, FIPS-compliant cryptographic primitives without external dependencies, reducing supply chain risk
- SHA-256 offers sufficient collision resistance (2^128 operations) for content addressing while maintaining broad compatibility and performance
- Stable JSON serialization ensures hash consistency across object property ordering variations, preventing false cache misses and integrity check failures
- Centralizing hash operations in core infrastructure (src/core/hash.ts) creates a single point of control for security updates and algorithm evolution

## Consequences

Positive:
- Eliminates external dependencies for core hashing operations, reducing attack surface and dependency maintenance burden
- Provides cryptographically secure content addressing with industry-standard SHA-256 algorithm
- Ensures deterministic hash generation through stable JSON serialization, improving cache hit rates and reducing false positives
- Centralizes hashing logic in core modules, enabling consistent security posture and simplified auditing

Negative:
- SHA-256 is computationally more expensive than non-cryptographic hash functions (e.g., xxHash, CityHash) for performance-critical paths
- Stable JSON serialization adds preprocessing overhead before hashing, impacting latency for large objects
- Node.js crypto module ties implementation to Node.js runtime, limiting portability to browser or edge environments without polyfills
- Future algorithm migration (e.g., to SHA-3) requires coordinated updates across all hash-dependent systems

## Alternatives

- Use non-cryptographic hash functions (xxHash, MurmurHash) for content addressing (rejected)
  Rejected because: Non-cryptographic hashes lack collision resistance guarantees necessary for security-sensitive integrity verification and are vulnerable to intentional collision attacks
  When valid: Acceptable for performance-critical internal caching where collision attacks are not a threat model concern
- Adopt external cryptographic libraries (crypto-js, forge) instead of Node.js crypto module (rejected)
  Rejected because: Introduces unnecessary external dependencies with additional supply chain risk when Node.js provides equivalent built-in functionality
  When valid: Required only when targeting non-Node.js environments (browser, Deno) where native crypto is unavailable
- Use SHA-3 (Keccak) as the standard hashing algorithm (deferred)
  Rejected because: SHA-256 provides sufficient security for current threat models; SHA-3 adoption can be reconsidered if SHA-2 vulnerabilities emerge
  When valid: Future migration if NIST recommends SHA-2 deprecation or specific compliance requirements mandate SHA-3

## Risks

- SHA-256 performance overhead may become bottleneck in high-throughput data processing pipelines
  Mitigation: Profile hash-intensive code paths; implement caching strategies; consider hybrid approach with non-cryptographic hashes for non-security-critical operations
  Owner: Engineering team with performance monitoring
- Future cryptographic vulnerabilities in SHA-256 could require emergency algorithm migration across all systems
  Mitigation: Abstract hashing behind stable API contracts (sha256 function); maintain version metadata with hashed content; design migration strategy for algorithm evolution
  Owner: Security team with architecture oversight
- Inconsistent JSON serialization implementations could break hash determinism across service boundaries
  Mitigation: Enforce use of centralized stableJson utility; add integration tests verifying cross-service hash consistency; document serialization contract
  Owner: Engineering team with API governance

## Implementation Notes

- Import hashing utilities exclusively from src/core/hash.ts to ensure consistent implementation across the codebase
- Always apply stableJson serialization to objects before passing to sha256 function to guarantee deterministic outputs
- Store algorithm metadata (e.g., 'sha256:' prefix) with hash outputs to support future algorithm migration without breaking existing hashes
- Consider implementing hash result caching for frequently hashed immutable data structures to amortize computational cost
- Document the security properties and appropriate use cases for SHA-256 hashing in developer guidelines to prevent misuse for password storage

## Continuation Context


Verify commands:
- grep -r "createHash" --include="*.ts" --include="*.js" | grep -v "src/core/hash.ts" | grep -v "test" || echo "All createHash usage centralized"
- grep -r "import.*sha256.*from.*['\"].*core/hash" --include="*.ts" --include="*.js" | wc -l
- node -e "const crypto = require('crypto'); const hash = crypto.createHash('sha256'); hash.update('test'); console.log(hash.digest('hex'));" | grep -q "^[a-f0-9]{64}$" && echo "SHA-256 verification passed"

Accept when:
- All cryptographic hash operations use the Node.js crypto module's createHash('sha256') exclusively
- The sha256 and stableJson functions are exported from src/core/hash.ts and imported by all consumers
- Test suite verifies hash determinism across multiple invocations with equivalent data structures
- No external hashing libraries (crypto-js, forge, etc.) are present in production dependencies

## Enforcement

- Verified by: Static analysis scanning for direct crypto.createHash usage outside core/hash.ts module
- Verified by: Dependency audits checking for external cryptographic libraries in package.json
- Verified by: Code review checklist requiring hash operations to use centralized utilities
- Verified by: Integration tests validating hash determinism across service boundaries
- Violation handling: CI pipeline fails if direct crypto.createHash usage detected outside approved modules
- Violation handling: Code review blocks merge if hashing logic bypasses core infrastructure
- Violation handling: Security team notified of violations for assessment of integrity impact
- Violation handling: Violations require refactoring to use centralized hash utilities before merge approval
- Exception process: Submit exception request with justification to security team via standard review process
- Exception process: Provide threat model analysis demonstrating why standard approach is insufficient
- Exception process: Document approved exceptions in ADR addendum with expiration date and review schedule
- Exception process: Exceptions require annual re-approval with updated security assessment