# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Cryptographic Hashing Operations

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires deterministic cryptographic hashing for data integrity verification and content addressing
- Node.js crypto module provides native SHA-256 implementation through createHash API
- JSON object serialization order affects hash output, requiring stable serialization to ensure consistent hash values across different runtime environments
- The pattern emerged in core infrastructure code (src/core/hash.ts) indicating foundational architectural significance

## Problem Statement

Without standardized cryptographic hashing and stable JSON serialization, the system cannot reliably generate consistent hash values for data integrity verification, content addressing, or deduplication across different execution contexts where object property ordering may vary.

## Decision

1. MUST: All cryptographic hashing operations MUST use SHA-256 algorithm via Node.js crypto.createHash('sha256')

## Policy Block

- MUST All cryptographic hashing operations MUST use SHA-256 algorithm via Node.js crypto.createHash('sha256')

In scope:
- All data integrity verification operations
- Content addressing and deduplication workflows
- Cryptographic hash generation for security-sensitive operations
- Any operation requiring deterministic hash values across runtime environments

Out of scope:
- Password hashing (requires specialized algorithms like bcrypt, argon2)
- HMAC operations requiring keyed hashing
- Legacy systems with existing hash implementations that cannot be migrated
- Performance-critical paths where cryptographic hashing overhead is prohibitive

Exceptions:
- EXC-001: Legacy integrations require MD5 or SHA-1 for compatibility with external systems

## Rationale

- SHA-256 provides industry-standard cryptographic strength (256-bit output) suitable for data integrity and content addressing without the vulnerabilities of older algorithms like MD5 or SHA-1
- Stable JSON serialization ensures hash consistency across JavaScript runtime environments where object property enumeration order may differ, preventing false mismatches in content comparison
- Centralizing hash utilities in core infrastructure (src/core/hash.ts) with public API contracts (sha256, stableJson) prevents implementation fragmentation and ensures consistent behavior
- The pattern's presence in core infrastructure with 90% confidence indicates established architectural practice worth codifying

## Consequences

Positive:
- Consistent hash values across all runtime environments and execution contexts enable reliable content addressing and deduplication
- Centralized implementation in core infrastructure reduces code duplication and maintenance burden
- Public API contracts provide clear integration points for all components requiring hashing functionality
- SHA-256 algorithm choice provides adequate security for non-password use cases while maintaining good performance

Negative:
- Stable JSON serialization adds computational overhead compared to native JSON.stringify
- SHA-256 is slower than non-cryptographic hash functions (e.g., xxHash) for performance-critical paths
- Centralized implementation creates a single point of failure if the core hash module has defects
- Migration of existing code using different hash algorithms or serialization approaches requires refactoring effort

## Alternatives

- Use non-cryptographic hash functions (e.g., xxHash, MurmurHash) for better performance (rejected)
  Rejected because: Non-cryptographic hashes lack collision resistance properties required for security-sensitive operations and data integrity verification
  When valid: Performance-critical paths where cryptographic properties are not required and collision risk is acceptable
- Allow multiple hash algorithms (SHA-256, SHA-512, BLAKE2) based on use case requirements (rejected)
  Rejected because: Multiple algorithms increase complexity, create interoperability issues, and fragment the codebase without clear benefit for current use cases
  When valid: Future requirements emerge that specifically need SHA-512 or BLAKE2 properties (e.g., quantum resistance considerations)
- Use native JSON.stringify without stable serialization (rejected)
  Rejected because: Property ordering inconsistencies across JavaScript engines would produce different hashes for semantically identical objects, breaking content addressing
  When valid: Never valid for hashing purposes; only acceptable for non-hash JSON serialization where ordering doesn't matter

## Risks

- SHA-256 may become cryptographically weak in future (e.g., quantum computing advances)
  Mitigation: Monitor NIST cryptographic standards and plan migration path to SHA-3 or post-quantum algorithms when necessary; design hash API to allow algorithm upgrades
  Owner: Security team
- Performance bottlenecks in high-throughput scenarios due to cryptographic hashing overhead
  Mitigation: Profile performance in production; implement caching strategies for frequently hashed values; consider non-cryptographic alternatives for specific performance-critical paths with security team approval
  Owner: Engineering team
- Stable JSON serialization implementation bugs could cause subtle hash inconsistencies
  Mitigation: Implement comprehensive test suite covering edge cases (nested objects, arrays, special characters, undefined values); use well-tested stable-stringify libraries rather than custom implementation
  Owner: Engineering team

## Implementation Notes

- Import hash utilities from src/core/hash.ts using the public API contracts: sha256() for hashing and stableJson() for serialization
- For object hashing, always serialize with stableJson() before passing to sha256() to ensure deterministic results
- Consider implementing hash result caching for immutable objects that are hashed frequently to reduce computational overhead
- Document the specific use case when adding new hash operations to ensure SHA-256 is appropriate (not password hashing, not performance-critical non-security paths)

## Continuation Context


Verify commands:
- grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts || echo 'No direct SHA-256 usage outside core module'
- grep -r "import.*sha256.*from.*core/hash" src/ | wc -l
- npm test -- --grep 'hash.*stable|stable.*json' 2>/dev/null || echo 'Verify stable JSON tests exist'

Accept when:
- All cryptographic hash operations use the centralized sha256 function from src/core/hash.ts
- No direct crypto.createHash calls exist outside the core hash module
- All object hashing operations use stableJson serialization before hashing
- Test suite validates hash consistency across different object property orderings

## Enforcement

- Verified by: Code review checklist requiring hash operations to use core/hash.ts API
- Verified by: Static analysis rules detecting direct crypto.createHash usage outside approved modules
- Verified by: CI pipeline tests validating hash consistency across test runs
- Verified by: Architecture review for new components using cryptographic operations
- Violation handling: CI build fails if direct crypto.createHash usage detected outside core/hash.ts
- Violation handling: Code review blocks merge if hash operations bypass standardized API
- Violation handling: Security team notified of violations in security-sensitive code paths
- Violation handling: Quarterly audit of hash usage patterns with remediation tracking
- Exception process: Submit exception request to security team with justification and use case details
- Exception process: Security team evaluates cryptographic requirements and approves/rejects within 3 business days
- Exception process: Approved exceptions documented in code comments with ticket reference and expiration date
- Exception process: Exceptions reviewed quarterly for continued validity and migration opportunities