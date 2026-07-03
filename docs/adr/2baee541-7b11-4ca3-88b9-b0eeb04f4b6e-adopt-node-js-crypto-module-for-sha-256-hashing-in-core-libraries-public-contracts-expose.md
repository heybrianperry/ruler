# Adopt Node.js crypto Module for SHA-256 Hashing in Core Libraries: Public Contracts Expose

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires deterministic hashing capabilities for data integrity verification and content addressing
- Core library modules need cryptographic hash functions that are stable across runtime environments
- The Node.js crypto module provides native, well-tested implementations of standard cryptographic algorithms including SHA-256
- A stable JSON serialization mechanism is needed to ensure consistent hash outputs for structured data objects

## Problem Statement

Core library code requires a standardized approach to generating cryptographic hashes for data integrity, content addressing, and deterministic identification. Without a consistent hashing strategy, different modules may implement incompatible hash functions, leading to verification failures and data integrity issues across system boundaries.

## Decision

1. MUST: Public API contracts MUST expose sha256 hashing functions for external consumers

## Policy Block

- MUST Public API contracts MUST expose sha256 hashing functions for external consumers

In scope:
- All core library modules under src/core/
- Public API contracts requiring hash-based identification
- Data integrity verification workflows
- Content-addressable storage implementations

Out of scope:
- Password hashing (use dedicated password hashing algorithms like bcrypt or argon2)
- HMAC or keyed hash operations (use crypto.createHmac instead)
- Non-cryptographic hash functions for performance-critical scenarios (e.g., hash tables)
- Third-party library internal implementations

## Rationale

- The evidence shows explicit use of crypto.createHash('sha256') in src/core/hash.ts, indicating an established pattern for cryptographic hashing in core libraries
- SHA-256 provides sufficient collision resistance for data integrity and content addressing use cases while being widely supported and standardized
- The stableJson serialization pattern ensures deterministic hash outputs for structured data, preventing hash mismatches due to property ordering variations
- Using the Node.js native crypto module avoids external dependencies and provides performance-optimized implementations

## Consequences

Positive:
- Consistent hash outputs across all core library modules enable reliable data integrity verification
- Public API contracts with sha256 functions provide standardized interfaces for external consumers
- Native crypto module usage eliminates external cryptographic dependencies and reduces supply chain risk
- Deterministic hashing via stableJson enables content-addressable storage and caching strategies

Negative:
- SHA-256 is computationally more expensive than non-cryptographic hash functions, potentially impacting performance in high-throughput scenarios
- Tight coupling to Node.js crypto module limits portability to browser or edge runtime environments without polyfills
- The stableJson serialization step adds processing overhead for structured data hashing operations

## Alternatives

- Use a third-party cryptographic library like crypto-js or noble-hashes (rejected)
  Rejected because: Introduces external dependencies with supply chain risk when Node.js provides native, well-audited implementations
  When valid: When targeting browser environments without Node.js crypto polyfills
- Use non-cryptographic hash functions like xxHash or MurmurHash (rejected)
  Rejected because: Non-cryptographic hashes lack collision resistance required for data integrity and security-sensitive use cases
  When valid: For performance-critical hash table implementations where collision resistance is not required
- Use SHA-512 or SHA-3 for stronger cryptographic guarantees (rejected)
  Rejected because: SHA-256 provides sufficient security for current use cases while offering better performance and wider compatibility
  When valid: When regulatory requirements or threat models demand stronger hash functions

## Risks

- Performance degradation in high-throughput scenarios due to SHA-256 computational cost
  Mitigation: Profile hash-intensive code paths and implement caching strategies for frequently hashed data. Consider batching operations where possible.
  Owner: engineering team
- Runtime environment incompatibility if code is deployed to browser or edge environments without crypto polyfills
  Mitigation: Document runtime requirements clearly and provide conditional imports or polyfills for non-Node.js environments
  Owner: engineering team
- Future SHA-256 deprecation or vulnerability discovery requiring algorithm migration
  Mitigation: Encapsulate hash implementation in dedicated modules to enable algorithm swapping without widespread code changes
  Owner: engineering team

## Implementation Notes

- Centralize hash utilities in src/core/hash.ts or similar modules to avoid duplication and enable consistent updates
- Implement stableJson serialization that sorts object keys deterministically before hashing structured data
- Export sha256 functions as part of public API contracts for external consumers requiring compatible hash generation
- Document the hash algorithm choice and serialization strategy in API documentation for consumers implementing compatible systems

## Continuation Context


Verify commands:
- grep -r "createHash('sha256')" src/core/ --include='*.ts'
- grep -r "from 'crypto'" src/core/ --include='*.ts'
- grep -r "stableJson" src/core/ --include='*.ts'

Accept when:
- All core library modules use crypto.createHash('sha256') for cryptographic hashing operations
- Public API contracts expose sha256 functions for external consumers
- Structured data hashing implementations use stableJson or equivalent deterministic serialization

## Enforcement

- Verified by: Static analysis scanning for crypto module imports and createHash usage patterns
- Verified by: Code review checklist verifying hash algorithm consistency
- Verified by: Unit tests validating deterministic hash outputs for identical inputs
- Violation handling: CI pipeline fails if non-standard hash implementations are detected in core libraries
- Violation handling: Code review blocks merge requests introducing alternative cryptographic hash libraries
- Violation handling: Automated linting rules flag createHash calls with algorithms other than sha256
- Exception process: Document specific use case requiring alternative hash algorithm or implementation
- Exception process: Obtain architecture review approval for exceptions with security implications
- Exception process: Add inline comments and exception registry entries explaining deviation rationale