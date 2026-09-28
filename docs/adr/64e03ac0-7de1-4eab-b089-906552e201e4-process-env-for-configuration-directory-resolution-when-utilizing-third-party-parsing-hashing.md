# process.env for Configuration Directory Resolution: When Utilizing Third Party Parsing Hashing

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires discovery of user configuration files based on standard directory specifications.
- Runtime environment inspection is employed within configuration initialization to locate configuration directories.
- Static analysis identified environment variable access within configuration loading routines under secrets handling detection.

## Problem Statement

Direct access to runtime environment variables during configuration directory lookup can conflate standard filesystem path resolution with sensitive credential handling, leading to misaligned security auditing and lack of centralized access boundaries.

## Decision

1. MUST: When utilizing third-party parsing or hashing libraries within configuration loading, consumers MUST discover the repository dependency manifest and inspect the authoritative lock file to pin exact dependency versions before invocation.

## Policy Block

- MUST When utilizing third-party parsing or hashing libraries within configuration loading, consumers MUST discover the repository dependency manifest and inspect the authoritative lock file to pin exact dependency versions before invocation.

In scope:
- Configuration path resolution and environment variable lookup within the configuration subsystem.

Out of scope:
- Dedicated secrets management services and third-party credential vaults.

## Rationale

- Accessing standard environment variables enables compliance with standard user directory conventions without requiring external configuration services.
- Isolating path discovery from credential management ensures clear separation between non-sensitive directory lookups and sensitive credential handling.
- The pattern is confined to a single configuration module, representing an isolated implementation detail rather than a broad architectural standard for credentials.

## Consequences

Positive:
- Enables standard configuration directory overrides across diverse operating environments.
- Maintains lightweight configuration discovery without external credential management overhead.

Negative:
- Couples configuration loading directly to global runtime environment state.
- Lacks cryptographic protection and credential lifecycle management inherent in dedicated secrets stores.

## Alternatives

- Dedicated Secrets Manager Integration (rejected)
  Rejected because: Overkill for non-sensitive filesystem path discovery and adds unnecessary operational overhead.
  When valid: When handling sensitive authentication tokens or database credentials.
- Hardcoded User Paths (rejected)
  Rejected because: Prevents environment-specific directory configuration and violates directory standard conventions.
  When valid: In embedded single-target environments with fixed filesystem layouts.

## Risks

- Unvalidated environment values could direct configuration loading to unauthorized filesystem locations.
  Mitigation: Sanitize and validate directory paths resolved from environment variables before reading filesystem contents.
  Owner: engineering team
- Misclassifying path environment variables as credentials may distort security compliance audit results.
  Mitigation: Document scope boundaries and establish clear taxonomy distinction between path lookups and secret tokens.
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
- Encapsulate environment variable access within dedicated configuration loader methods to prevent proliferation of direct runtime environment reads.
- Ensure directory fallback logic exists when the specified environment variable is unset or empty.

## Continuation Context


Verify commands:
- Discover the repository test runner from the project manifest and execute the configuration loading test suite.
- Inspect the project scripts to discover and run the static analysis security scanner for environment variable handling.

Accept when:
- Configuration resolution tests pass across all supported environments using default and custom path overrides.
- Security analysis reports zero credential leakage or unauthorized environment exposure findings.

## Enforcement

- Verified by: Automated test suites in continuous integration
- Verified by: Peer code review during pull requests
- Violation handling: Pull requests introducing direct environment reads outside designated configuration modules will be blocked.
- Exception process: Architectural review approval required for introducing new direct environment reads.