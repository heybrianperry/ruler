# Standardize SHA-256 Hashing via Node.js Crypto Module for Deterministic Content Fingerprinting: Modules Cache Hash

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires deterministic content fingerprinting for data integrity verification, deduplication, or content-addressable storage patterns
- SHA-256 provides cryptographic-strength collision resistance suitable for content identification without requiring external dependencies beyond Node.js standard library
- The pattern is observed in src/core/hash.ts, which exposes sha256 and stableJson as public API contracts, indicating this is a foundational utility used across the system
- Using Node.js built-in crypto module eliminates third-party dependency risk while maintaining FIPS 140-2 compliance where Node.js is compiled with OpenSSL FIPS module

## Problem Statement

The system needs a consistent, cryptographically sound method for generating content fingerprints that remain stable across serialization boundaries, without introducing external cryptographic library dependencies or inconsistent hashing implementations across modules.

## Decision

1. MAY: Modules MAY cache hash results for immutable content to avoid redundant computation

## Policy Block

- MAY Modules MAY cache hash results for immutable content to avoid redundant computation

## Rationale

- Evidence shows explicit use of crypto.createHash('sha256') in src/core/hash.ts with public API contracts (sha256, stableJson), indicating intentional standardization of hashing approach
- SHA-256 provides 256-bit collision resistance (2^128 security level) sufficient for content addressing without the performance overhead of SHA-512 or SHA-3
- Centralizing hash logic in src/core/hash.ts with stableJson normalization ensures consistent fingerprints across different serialization orders or whitespace variations
- Node.js crypto module is maintained as part of the Node.js security release process, reducing supply chain risk compared to third-party cryptographic libraries

## Consequences

Positive:
- Deterministic content fingerprinting enables reliable deduplication, caching, and content-addressable storage patterns
- Zero external dependencies for cryptographic hashing reduces attack surface and simplifies security audits
- Centralized hash implementation in src/core/hash.ts ensures consistent behavior across all consuming modules
- SHA-256 provides sufficient collision resistance for content identification while maintaining acceptable performance characteristics

Negative:
- SHA-256 computation adds CPU overhead for large content payloads compared to non-cryptographic hash functions like xxHash or MurmurHash
- Tight coupling to Node.js crypto module limits portability to browser environments without polyfills or WebCrypto API adaptation
- stableJson serialization may introduce performance bottlenecks for deeply nested or large object structures requiring normalization before hashing
- Future migration to alternative hash algorithms (e.g., BLAKE3) requires coordinated updates across all consumers of src/core/hash.ts API

## Alternatives

- Use non-cryptographic hash functions (xxHash, MurmurHash) for content fingerprinting (rejected)
  Rejected because: Non-cryptographic hashes lack collision resistance guarantees required for content-addressable storage and security-sensitive deduplication scenarios
  When valid: Valid for performance-critical scenarios where collision resistance is not required (e.g., in-memory hash tables, bloom filters)
- Adopt third-party cryptographic library (crypto-js, noble-hashes) instead of Node.js crypto module (rejected)
  Rejected because: Introduces external dependency with supply chain risk and maintenance burden when Node.js crypto module provides equivalent functionality
  When valid: Valid for browser-only environments where Node.js crypto module is unavailable and WebCrypto API is insufficient
- Use SHA-512 or SHA-3 for enhanced security margin (rejected)
  Rejected because: SHA-256 provides sufficient collision resistance (2^128 security level) for content addressing; SHA-512/SHA-3 adds computational overhead without meaningful security benefit for this use case
  When valid: Valid for cryptographic signatures or scenarios requiring 256-bit security level (e.g., post-quantum resistance considerations)

## Risks

- Performance degradation when hashing large content payloads or high-frequency hashing operations
  Mitigation: Implement caching layer for immutable content hashes; profile hash operations and consider streaming hash updates for large payloads; benchmark against performance budgets
  Owner: engineering team
- stableJson serialization may not handle all edge cases (circular references, non-serializable types, custom toJSON implementations)
  Mitigation: Document stableJson limitations; implement validation for supported input types; add error handling for serialization failures with clear error messages
  Owner: engineering team
- Future cryptographic vulnerabilities in SHA-256 could require migration to alternative hash algorithms
  Mitigation: Encapsulate hash algorithm selection behind src/core/hash.ts API; version hash outputs with algorithm identifier if persistence is required; monitor NIST cryptographic standards updates
  Owner: engineering team

## Implementation Notes

- Import sha256 and stableJson from src/core/hash.ts rather than directly invoking crypto.createHash to maintain centralized control over hash implementation
- For large content payloads, consider streaming hash updates using hash.update() in chunks rather than loading entire content into memory before hashing
- Document expected input types for stableJson normalization and handle serialization errors gracefully with fallback strategies or clear error messages
- When persisting hash values, consider prefixing with algorithm identifier (e.g., 'sha256:') to enable future algorithm migration without breaking existing stored hashes

## Continuation Context


Verify commands:
- grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts | wc -l | grep -q '^0$'
- grep -r "from ['\"].*hash['\"]" src/ | grep -E "(sha256|stableJson)" | wc -l
- node -e "const {sha256, stableJson} = require('./src/core/hash'); console.assert(sha256(stableJson({b:2,a:1})) === sha256(stableJson({a:1,b:2})))"

Accept when:
- No direct crypto.createHash('sha256') invocations exist outside src/core/hash.ts
- All modules requiring content hashing import sha256 and/or stableJson from src/core/hash.ts
- stableJson produces identical serialization for equivalent objects regardless of property order

## Enforcement

- Verified by: Static analysis via grep/ripgrep to detect direct crypto.createHash usage outside src/core/hash.ts
- Verified by: Unit tests validating stableJson deterministic serialization across property order variations
- Verified by: Code review checklist requiring use of src/core/hash.ts API for all content fingerprinting operations
- Violation handling: CI pipeline fails if direct crypto.createHash invocations detected outside src/core/hash.ts
- Violation handling: Code review blocks merge if content hashing bypasses src/core/hash.ts API without documented exception
- Violation handling: Automated linting rules flag non-compliant hash usage with remediation guidance
- Exception process: Document technical justification for exception (e.g., performance-critical path requiring non-cryptographic hash)
- Exception process: Obtain approval from security and architecture review board
- Exception process: Add inline comment with exception rationale and link to approval decision record