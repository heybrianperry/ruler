# XDG_CONFIG_HOME Environment Configuration Resolution: Runtime Modules Not Access Process Env

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Global configuration resolution across command-line execution, filesystem operations, and configuration loading requires deterministic directory locations across diverse operating systems and user environments.
- Direct inspection of process.env.XDG_CONFIG_HOME provides standard directory hierarchy compliance for user-scoped configuration storage before falling back to default user home directories.
- Multiple modules across the runtime execution boundary independently reference process.env.XDG_CONFIG_HOME alongside platform path utilities, introducing divergence risks if environment variable precedence or fallback resolution is implemented inconsistently.

## Problem Statement

Runtime configuration discovery requires a deterministic, platform-standard mechanism to resolve user global configuration directories. When multiple runtime modules directly query environment variables without unified resolution rules, configuration discovery becomes prone to path inconsistencies, improper fallbacks when the environment variable is unset or empty, and security risks associated with unvalidated path traversal.

## Decision

1. MUST_NOT: Runtime modules MUST_NOT access process.env.XDG_CONFIG_HOME directly outside of dedicated configuration resolution modules to prevent divergent path resolution semantics.

## Policy Block

- MUST_NOT Runtime modules MUST_NOT access process.env.XDG_CONFIG_HOME directly outside of dedicated configuration resolution modules to prevent divergent path resolution semantics.

In scope:
- Resolution of global user configuration directories during runtime initialization.
- Command-line execution handlers and configuration loaders querying environment settings.

Out of scope:
- Local project-scoped configuration files located relative to repository root directories.
- In-memory or ephemeral configuration passed directly through test harnesses.

Exceptions:
- EXC-37-001: Isolated test environments explicitly require sandboxed temporary directories without reading the host user configuration.

## Rationale

- Adopting process.env.XDG_CONFIG_HOME aligns runtime directory resolution with standard desktop and server configuration hierarchy conventions, giving users full control over configuration placement.
- Centralizing environment-based configuration discovery prevents duplicate path resolution logic and eliminates discrepancies between command-line handlers, filesystem utilities, and configuration loaders.
- Requiring absolute path normalization and graceful error handling mitigates filesystem permission failures and path traversal risks when reading external environment variables.

## Consequences

Positive:
- Establishes deterministic, standards-compliant configuration directory discovery across heterogeneous platforms.
- Enables users and automated execution environments to redirect configuration storage seamlessly through a single environment variable.
- Eliminates divergent path resolution and inconsistent fallback behavior across command-line handlers, filesystem utilities, and configuration loaders.

Negative:
- Introduces runtime dependency on process environment variables that requires explicit mocking or isolation in automated integration test suites.
- Requires path validation and normalization logic to handle malformed, relative, or whitespace-only environment variable values safely.

## Alternatives

- Hardcode configuration directory paths exclusively to the operating system user home directory. (rejected)
  Rejected because: Disregards platform directory standards and prevents containerized or unprivileged environments from overriding configuration storage locations.
  When valid: Single-platform systems with no requirement for user-level environment customization.
- Allow every runtime module to query and resolve process.env.XDG_CONFIG_HOME independently with local fallback logic. (rejected)
  Rejected because: Introduces semantic divergence where command-line handlers, filesystem utilities, and configuration loaders may resolve differing configuration paths under identical environment conditions.
  When valid: Ad-hoc single-file utilities with no shared runtime or configuration architecture.

## Risks

- An empty or whitespace-only process.env.XDG_CONFIG_HOME string could resolve to an unintended relative directory.
  Mitigation: Validate that process.env.XDG_CONFIG_HOME is non-empty, contains valid characters, and resolves to an absolute path before adopting it as the configuration base.
  Owner: Core Engineering Team
- Permission denials or filesystem errors when reading from the resolved global configuration directory could crash the runtime.
  Mitigation: Implement structured error handling and logging around directory checks to report accessibility issues without unhandled process termination.
  Owner: Core Engineering Team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Encapsulate global configuration directory resolution within a single reusable utility function that checks process.env.XDG_CONFIG_HOME, applies fallback to standard user home directory paths, and normalizes the resulting path.
- Ensure all consumer modules across execution boundaries import the shared resolution utility rather than reading process.env directly.

## Continuation Context


Verify commands:
- Discover the test runner from the project repository manifest and run the test suite verifying configuration resolution against custom and fallback environment variables.
- Discover the static analysis tool from the project repository manifest and verify that direct references to process.env.XDG_CONFIG_HOME are restricted to designated configuration utility modules.

Accept when:
- Test suites pass verifying that setting process.env.XDG_CONFIG_HOME redirects global configuration discovery to the specified directory path.
- Test suites pass verifying that unsetting or clearing process.env.XDG_CONFIG_HOME correctly falls back to the default operating system user directory.
- Static analysis checks confirm that direct access to process.env.XDG_CONFIG_HOME is restricted to designated configuration resolution modules.

## Enforcement

- Verified by: Continuous integration test suites verifying configuration resolution with and without environment variable overrides.
- Verified by: Automated static analysis and code review checks verifying that environment variable access is encapsulated within dedicated modules.
- Violation handling: Pull requests containing unencapsulated direct access to process.env.XDG_CONFIG_HOME are blocked until refactored to use the central configuration utility.
- Violation handling: Builds fail if path resolution tests fail or if unvalidated environment paths cause unhandled filesystem exceptions.
- Exception process: Submit an architectural review request detailing why an external entry point requires direct environment variable access without the central configuration utility.