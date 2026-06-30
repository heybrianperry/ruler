# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Object Structures Requiring

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires deterministic cryptographic hashing for data integrity verification and content addressing
- The Node.js crypto module provides native SHA-256 hashing capabilities without external dependencies
- JSON object serialization order affects hash output, requiring stable serialization to ensure consistent hash values across different JavaScript runtime environments
- The pattern emerged in core infrastructure code (src/core/hash.ts) indicating foundational architectural significance

## Problem Statement

Without standardized cryptographic hashing and stable JSON serialization, the system cannot reliably generate consistent hash values for data structures, leading to potential integrity verification failures, cache invalidation issues, and content addressing inconsistencies across different execution contexts.

## Decision

1. MUST: All object structures requiring hashing MUST be serialized using stable JSON serialization that produces deterministic output regardless of property insertion order

## Policy Block

- MUST All object structures requiring hashing MUST be serialized using stable JSON serialization that produces deterministic output regardless of property insertion order

In scope:
- All data integrity verification operations
- Content addressing and deduplication systems
- Cache key generation requiring deterministic hashing
- API contracts requiring stable hash outputs

Out of scope:
- Password hashing (use bcrypt, argon2, or similar)
- HMAC operations requiring keyed hashing
- Random number generation
- Non-cryptographic hash functions for performance-critical operations where collision resistance is not required

## Rationale

- SHA-256 provides sufficient collision resistance and cryptographic strength for data integrity use cases while being widely supported and performant
- Stable JSON serialization ensures hash consistency across different JavaScript engines, runtime versions, and object construction patterns
- Centralizing hash operations in core infrastructure (src/core/hash.ts) establishes a single source of truth and prevents algorithm fragmentation
- The Node.js crypto module is a standard library component requiring no external dependencies, reducing supply chain risk

## Consequences

Positive:
- Deterministic hash values enable reliable content addressing and cache invalidation strategies
- Centralized implementation reduces code duplication and ensures consistent security properties
- Native crypto module usage eliminates external cryptographic library dependencies
- Public API contracts (sha256, stableJson) provide clear integration points for consumers

Negative:
- Stable JSON serialization adds computational overhead compared to native JSON.stringify
- SHA-256 is slower than non-cryptographic hash functions for performance-critical paths
- Changing the hashing algorithm in the future requires migration of all existing hash-dependent data
- Object serialization may fail or produce unexpected results for complex types (functions, symbols, circular references)

## Alternatives

- Use SHA-1 for hashing operations (rejected)
  Rejected because: SHA-1 is cryptographically broken with demonstrated collision attacks, making it unsuitable for security-sensitive operations
  When valid: Never for cryptographic purposes; only acceptable for legacy compatibility with external systems explicitly requiring SHA-1
- Use non-cryptographic hash functions (e.g., xxHash, MurmurHash) (rejected)
  Rejected because: Non-cryptographic hashes lack collision resistance required for data integrity verification and security contexts
  When valid: Performance-critical operations where collision resistance is not required (e.g., hash tables, bloom filters)
- Use SHA-512 for stronger security guarantees (rejected)
  Rejected because: SHA-256 provides sufficient security for current use cases while offering better performance and smaller hash outputs
  When valid: Systems requiring 256-bit collision resistance or compliance requirements mandating SHA-512

## Risks

- Stable JSON serialization may not handle all edge cases (circular references, special types) leading to runtime errors
  Mitigation: Implement comprehensive input validation and error handling in stableJson function; document supported input types clearly
  Owner: engineering team
- Performance degradation in high-throughput scenarios due to serialization and hashing overhead
  Mitigation: Profile critical paths and implement caching strategies for frequently hashed values; consider memoization for immutable objects
  Owner: engineering team
- Future cryptographic vulnerabilities in SHA-256 may require algorithm migration
  Mitigation: Abstract hash algorithm selection behind the public API to enable future algorithm changes without widespread code modifications
  Owner: engineering team

## Implementation Notes

- Import hash utilities from src/core/hash.ts using the public API contracts (sha256, stableJson) rather than implementing custom hashing logic
- For objects requiring hashing, always apply stableJson serialization before passing to sha256 to ensure deterministic output
- Consider implementing hash result caching for immutable objects that are hashed frequently to amortize serialization costs
- Document the expected input types and constraints for hash functions to prevent misuse with unsupported data structures

## Continuation Context


Verify commands:
- grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts
- grep -r "require('crypto')" src/ | grep -v src/core/hash.ts
- npm test -- --grep 'hash|sha256|stableJson'

Accept when:
- No direct usage of crypto.createHash outside of src/core/hash.ts (all hashing goes through centralized API)
- All hash-related tests pass demonstrating deterministic output for identical inputs
- Public API exports sha256 and stableJson functions that are imported by consumers

## Enforcement

- Verified by: Code review checking for direct crypto module usage outside core infrastructure
- Verified by: Automated linting rules detecting createHash usage outside approved modules
- Verified by: Unit tests validating hash determinism and stable JSON serialization properties
- Violation handling: Pull requests introducing direct crypto usage outside src/core/hash.ts are rejected during code review
- Violation handling: CI pipeline fails if grep verification commands detect violations
- Violation handling: Existing violations are tracked as technical debt items for refactoring
- Exception process: Document the specific use case requiring exception in ADR amendment or inline code comments
- Exception process: Obtain approval from security team for cryptographic operations outside standard patterns
- Exception process: Add exception to verification command exclusion list with justification