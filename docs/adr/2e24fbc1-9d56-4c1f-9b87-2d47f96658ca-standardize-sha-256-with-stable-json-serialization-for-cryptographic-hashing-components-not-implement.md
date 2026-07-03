# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Components Not Implement

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires deterministic cryptographic hashing for data integrity verification and content addressing
- Node.js crypto module provides native SHA-256 implementation through createHash API
- Object serialization order affects hash output, requiring stable JSON serialization to ensure consistency
- Public API contracts expose sha256 and stableJson functions, establishing them as architectural primitives

## Problem Statement

Without standardized cryptographic hashing and deterministic serialization, different parts of the system may produce inconsistent hash values for equivalent data structures, breaking content addressing, cache invalidation, and integrity verification mechanisms.

## Decision

1. MUST_NOT: Components MUST NOT implement custom hashing logic outside the core hash module

## Policy Block

- MUST_NOT Components MUST NOT implement custom hashing logic outside the core hash module

In scope:
- All data integrity verification operations
- Content addressing and deduplication
- Cache key generation
- Cryptographic fingerprinting of configuration or state

Out of scope:
- Password hashing (use bcrypt, argon2, or similar)
- HMAC operations requiring secret keys
- Digital signatures requiring asymmetric cryptography
- Random number generation

Exceptions:
- EX-001: Legacy systems require MD5 or SHA-1 for backward compatibility

## Rationale

- SHA-256 provides sufficient collision resistance for content addressing while being widely supported and performant
- Stable JSON serialization eliminates non-determinism from object key ordering, ensuring identical objects produce identical hashes
- Centralizing hash operations in core library prevents algorithm fragmentation and simplifies security audits
- Public API contracts establish sha256 and stableJson as architectural primitives that other modules can depend on

## Consequences

Positive:
- Deterministic hashing enables reliable content addressing and cache invalidation
- Centralized implementation simplifies security updates and algorithm migration
- Public API contracts provide clear integration points for dependent modules
- SHA-256 offers strong collision resistance suitable for integrity verification

Negative:
- SHA-256 is slower than non-cryptographic hashes like xxHash for performance-critical paths
- Stable JSON serialization adds computational overhead for complex objects
- Dependency on Node.js crypto module limits portability to browser environments without polyfills
- Single algorithm choice may require migration effort if SHA-256 becomes deprecated

## Alternatives

- Use non-cryptographic hash functions (xxHash, MurmurHash) for performance (rejected)
  Rejected because: Non-cryptographic hashes lack collision resistance required for security-sensitive integrity verification
  When valid: Performance-critical paths where collision resistance is not required (e.g., in-memory hash tables)
- Use SHA-3 (Keccak) for future-proof cryptographic strength (rejected)
  Rejected because: SHA-3 has lower ecosystem support and performance compared to SHA-256, with no practical security advantage for current use cases
  When valid: When NIST specifically recommends SHA-3 for compliance requirements
- Allow multiple hash algorithms with runtime selection (rejected)
  Rejected because: Algorithm diversity increases complexity and creates hash incompatibility between system components
  When valid: When supporting multiple external systems with different hash requirements

## Risks

- SHA-256 performance bottleneck in high-throughput scenarios
  Mitigation: Profile hash operations and implement caching for frequently hashed objects; consider worker threads for CPU-intensive hashing
  Owner: Engineering team
- Stable JSON serialization may not handle all edge cases (circular references, special types)
  Mitigation: Document supported data types; implement validation to reject unsupported structures; add comprehensive test coverage
  Owner: Core library maintainers
- Future SHA-256 deprecation requires system-wide migration
  Mitigation: Version hash outputs with algorithm identifier; design hash storage to support multiple algorithms; monitor NIST cryptographic standards
  Owner: Security team

## Implementation Notes

- Import sha256 and stableJson from src/core/hash.ts rather than implementing custom hashing
- For objects, always serialize with stableJson before passing to sha256 to ensure determinism
- Consider caching hash results for immutable objects to avoid redundant computation
- Document hash algorithm version in persistent storage schemas to support future migration

## Continuation Context


Verify commands:
- grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts
- grep -r "require('crypto')" src/ | grep -v src/core/hash.ts | grep -v test
- node -e "const {sha256, stableJson} = require('./src/core/hash'); console.log(sha256(stableJson({b:2,a:1})) === sha256(stableJson({a:1,b:2})))"

Accept when:
- No direct crypto.createHash('sha256') calls exist outside src/core/hash.ts
- All modules requiring hashing import from src/core/hash.ts
- Stable JSON serialization produces identical hashes for equivalent objects regardless of key order

## Enforcement

- Verified by: Static analysis scanning for direct crypto module usage outside core hash module
- Verified by: Code review checklist requiring hash module imports
- Verified by: Unit tests verifying hash determinism across object key orderings
- Violation handling: CI pipeline fails on detection of crypto usage outside core module
- Violation handling: Code review blocks merge until hash operations use core API
- Violation handling: Security audit flags non-standard hash implementations for remediation
- Exception process: Submit exception request to security team with justification
- Exception process: Document alternative algorithm choice and rationale in ADR addendum
- Exception process: Implement monitoring for exception usage and schedule review