# Adoption of Node.js Crypto Module for SHA256 Hashing: Before Implementing Updating Code That Uses

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- The project requires cryptographic hashing capabilities for data protection.
- The `crypto` module is a built-in Node.js module providing cryptographic functionalities.
- SHA256 is a widely accepted secure hashing algorithm.

## Problem Statement

The project needs a standardized and secure method for cryptographic hashing to ensure data integrity and security, specifically for operations requiring a robust one-way hash function.

## Decision

1. MUST: Before implementing or updating code that uses the `crypto` module, developers MUST discover the ecosystem's lock file and resolve the exact locked version of Node.js and its built-in modules to ensure API compatibility and security patch levels.

## Policy Block

- MUST Before implementing or updating code that uses the `crypto` module, developers MUST discover the ecosystem's lock file and resolve the exact locked version of Node.js and its built-in modules to ensure API compatibility and security patch levels.

In scope:
- Code requiring cryptographic hashing for data protection.

Out of scope:
- Operations not requiring cryptographic hashing, or those explicitly needing different cryptographic primitives (e.g., encryption, signing) not covered by SHA256 hashing.

## Rationale

- The `crypto` module is a native Node.js module, providing a performant and secure implementation of cryptographic primitives.
- Using a standardized module like `crypto` reduces the risk of introducing vulnerabilities from third-party or custom implementations.
- SHA256 is a strong, collision-resistant hash function suitable for various data protection scenarios.

## Consequences

Positive:
- Consistent and secure cryptographic hashing across the codebase.
- Reduced dependency on external libraries for core cryptographic functions.
- Leverages well-tested and maintained native Node.js capabilities.

Negative:
- Developers must be familiar with the `crypto` module's API.
- Changes in Node.js `crypto` module behavior or security vulnerabilities could impact the project.

## Alternatives

- Use a third-party cryptographic library (e.g., `js-sha256`). (rejected)
  Rejected because: Introduces an additional external dependency and potential supply chain risks when a native, secure alternative is available.
  When valid: For environments where Node.js `crypto` module is not available or specific features not provided by `crypto` are required.

## Risks

- Misuse of cryptographic primitives leading to security vulnerabilities.
  Mitigation: Provide clear documentation and code examples for correct usage; conduct code reviews focusing on security.
  Owner: Engineering Team
- Future vulnerabilities discovered in the `crypto` module.
  Mitigation: Regularly update Node.js to the latest secure versions; monitor security advisories.
  Owner: Engineering Team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Ensure proper handling of input data to the hashing function to prevent common security pitfalls (e.g., timing attacks, side-channel leaks).
- Consider using a key derivation function (KDF) like PBKDF2 or scrypt from the `crypto` module for password hashing, rather than direct SHA256, to add salt and iteration count.

## Continuation Context


Verify commands:
- Inspect the project's dependency manifest to confirm the presence of Node.js as a runtime dependency.
- Examine relevant source files for calls to cryptographic hashing functions.
- Run the project's test suite to ensure cryptographic operations are correctly implemented.

Accept when:
- The project's dependency manifest explicitly lists Node.js as a runtime.
- Source code demonstrates usage of the `crypto` module's `createHash('sha256')` function for hashing.
- All security-related tests pass without errors or warnings.

## Enforcement

- Verified by: Automated CI checks, code reviews by security-aware developers.
- Violation handling: Code failing CI checks will block merges; code review comments will require remediation before approval.
- Exception process: Exceptions require explicit approval from a lead architect or security officer, documented with a clear rationale.