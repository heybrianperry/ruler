# Cryptographic Digest Generation via crypto createHash: Components Not Instantiate Custom Unverified Hashing

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Application components require consistent, collision-resistant cryptographic digests for data integrity and identity verification.
- Platform standard crypto capabilities provide native digest generation functions without external dependencies.
- Centralizing hashing operations within core module abstractions prevents divergent digest algorithms across the system.

## Problem Statement

Data integrity and hashing operations require a standardized cryptographic hashing mechanism to prevent algorithm fragmentation, insecure hash function choices, and inconsistent digest representations across modules.

## Decision

1. MUST_NOT: Components MUST NOT instantiate custom or unverified hashing implementations when cryptographic digest generation is required.

## Policy Block

- MUST_NOT Components MUST NOT instantiate custom or unverified hashing implementations when cryptographic digest generation is required.

In scope:
- Modules performing cryptographic digest computation or data integrity verification.

Out of scope:
- Non-cryptographic hashing scenarios where fast hash performance takes precedence over collision resistance and security guarantees.

## Rationale

- Standardizing on the crypto module createHash method ensures uniform cryptographic guarantees without introducing external third-party dependencies.
- The sha256 algorithm provides strong collision resistance and wide compatibility across infrastructure components.

## Consequences

Positive:
- Guarantees consistent digest format and cryptographic algorithm strength across data protection workflows.
- Eliminates third-party supply chain overhead by relying on standard platform crypto primitives.

Negative:
- Restricts hashing implementations to synchronous or platform-provided crypto streams, requiring adapter wrappers for differing execution environments.
- Imposes fixed algorithm computational overhead that may be sub-optimal for non-security checksum needs.

## Alternatives

- Third-party hashing libraries (rejected)
  Rejected because: Introduces external dependency risk and maintenance overhead when platform runtime primitives provide certified implementations.
  When valid: When targeting runtimes lacking standard cryptographic built-in modules.
- Non-cryptographic hashing functions (rejected)
  Rejected because: Does not satisfy security and collision resistance requirements for secure data protection.
  When valid: In-memory hash table lookups where cryptographic collision resistance is unnecessary.

## Risks

- Runtime environment differences may affect availability or behavior of standard crypto module APIs.
  Mitigation: Encapsulate crypto calls behind core abstraction layers with automated verification test suites across all target environments.
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
- Isolate cryptographic hashing functions behind reusable core helper signatures to abstract invocation patterns and input encoding.

## Continuation Context


Verify commands:
- Discover the project verification script from the repository manifest and execute the test suite covering core cryptographic modules.
- Discover and run the static analysis linting commands configured in the repository to verify that direct hash instantiation conforms to approved rules.

Accept when:
- Cryptographic digest generation tests pass successfully using the designated algorithm.
- Static analysis validates that hashing operations route through the approved crypto interfaces.

## Enforcement

- Verified by: Automated test suites discovered in the repository verification pipeline.
- Verified by: Static analysis checks configured in the repository.
- Verified by: Peer code review for pull requests modifying core cryptographic operations.
- Violation handling: Pull request verification failures block merging until unauthorized hashing patterns are replaced with approved crypto createHash invocations.
- Exception process: Exceptions require security architecture approval documented in an architectural exception record justifying non-standard digest algorithms.