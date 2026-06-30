# Use Node.js crypto Module for SHA-256 Hashing in Core Libraries: Core Hashing Utilities

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires deterministic hashing capabilities for data integrity verification and content addressing
- Core library modules need cryptographic hash functions that are stable across different runtime environments
- The Node.js crypto module provides native, well-tested implementations of standard cryptographic algorithms without external dependencies
- SHA-256 is selected as the standard hashing algorithm due to its widespread adoption, collision resistance, and balance between security and performance

## Problem Statement

Core library code requires a consistent, secure, and performant method for generating cryptographic hashes of data structures and content. The solution must produce stable, deterministic outputs suitable for content addressing, integrity verification, and deduplication while avoiding external dependencies that could introduce supply chain risks or version conflicts.

## Decision

1. MUST_NOT: Core hashing utilities MUST NOT depend on external cryptographic libraries when Node.js crypto provides equivalent functionality

## Policy Block

- MUST_NOT Core hashing utilities MUST NOT depend on external cryptographic libraries when Node.js crypto provides equivalent functionality

In scope:
- All core library modules requiring cryptographic hashing
- Public API contracts that expose hashing functionality
- Content addressing and integrity verification systems
- Data structures requiring deterministic serialization before hashing

Out of scope:
- Password hashing or key derivation functions (use bcrypt, scrypt, or PBKDF2)
- HMAC or message authentication codes (separate decision required)
- Application-level code outside core libraries (may use alternative approaches with justification)
- Legacy code with existing hash implementations (migration path to be defined separately)

## Rationale

- The Node.js crypto module is a built-in, well-maintained implementation that eliminates external dependencies and reduces supply chain attack surface
- SHA-256 provides sufficient collision resistance for content addressing while maintaining broad compatibility and good performance characteristics
- Stable JSON serialization combined with SHA-256 ensures deterministic hashing across different execution contexts, which is critical for distributed systems and caching
- Exposing sha256 and stableJson as public contracts creates a consistent interface for hash-based operations throughout the codebase

## Consequences

Positive:
- Reduced dependency footprint and lower supply chain risk by using native Node.js modules
- Deterministic hashing enables reliable content addressing, deduplication, and cache invalidation
- Consistent cryptographic primitives across core libraries improve maintainability and reduce cognitive load
- SHA-256 is widely supported and understood, facilitating interoperability with external systems

Negative:
- SHA-256 is slower than non-cryptographic hash functions like xxHash or MurmurHash for non-security use cases
- Tight coupling to Node.js crypto module may complicate future migration to other JavaScript runtimes
- Stable JSON serialization adds computational overhead compared to hashing raw binary data
- Limited flexibility to adopt newer hash algorithms without breaking existing content addresses

## Alternatives

- Use third-party cryptographic libraries like crypto-js or noble-hashes (rejected)
  Rejected because: Introduces external dependencies with supply chain risks and version management overhead when Node.js crypto provides equivalent functionality
  When valid: When targeting browser environments where Node.js crypto is unavailable
- Use non-cryptographic hash functions like xxHash for performance (rejected)
  Rejected because: Non-cryptographic hashes lack collision resistance required for security-sensitive operations and content integrity verification
  When valid: For internal data structures where collision resistance is not required and performance is critical
- Use SHA-512 or SHA-3 for stronger security guarantees (rejected)
  Rejected because: SHA-256 provides sufficient security for current use cases while offering better performance and wider compatibility
  When valid: When regulatory requirements or threat models specifically mandate stronger hash functions

## Risks

- Future quantum computing advances may weaken SHA-256 collision resistance
  Mitigation: Monitor NIST post-quantum cryptography standards and plan migration path to quantum-resistant hash functions when standardized
  Owner: Security team
- Stable JSON serialization may not handle all edge cases consistently across JavaScript engines
  Mitigation: Implement comprehensive test suite covering edge cases (undefined, symbols, circular references) and document serialization behavior
  Owner: Core library team
- Performance bottlenecks may emerge when hashing large data structures or high-frequency operations
  Mitigation: Profile hash operations in production, implement caching strategies, and consider streaming hash updates for large inputs
  Owner: Engineering team

## Implementation Notes

- Import createHash from 'crypto' module and instantiate with 'sha256' algorithm parameter
- Implement stableJson utility that sorts object keys recursively before JSON.stringify to ensure deterministic serialization
- Export both sha256 and stableJson functions as public API contracts from core hash module
- Document the stable serialization rules including handling of undefined values, functions, and non-serializable types
- Consider implementing streaming hash updates using hash.update() for large inputs to reduce memory pressure

## Continuation Context


Verify commands:
- grep -r "createHash('sha256')" src/core/ | grep -v node_modules
- grep -r "from 'crypto'" src/core/hash.ts
- grep -E "export.*(sha256|stableJson)" src/core/hash.ts

Accept when:
- The core hash module imports createHash from Node.js crypto module
- SHA-256 is used via createHash('sha256') for cryptographic hashing operations
- Public API exports include both sha256 and stableJson functions as documented contracts

## Enforcement

- Verified by: Automated code review checks for crypto module usage in core libraries
- Verified by: Static analysis tools scanning for third-party cryptographic dependencies in core modules
- Verified by: Unit tests validating deterministic hash output across different inputs and execution contexts
- Violation handling: CI pipeline fails if core modules import external cryptographic libraries without documented exception
- Violation handling: Code review blocks merges that introduce non-standard hashing approaches in core libraries
- Violation handling: Automated alerts notify security team of cryptographic dependency changes
- Exception process: Document technical justification for alternative approach including security and performance analysis
- Exception process: Obtain approval from security team and core library maintainers
- Exception process: Record exception in ADR amendments with expiration date and migration plan