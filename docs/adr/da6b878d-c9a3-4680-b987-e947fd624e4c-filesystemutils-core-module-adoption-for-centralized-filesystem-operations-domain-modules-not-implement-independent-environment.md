# FileSystemUtils Core Module Adoption for Centralized Filesystem Operations: Domain Modules Not Implement Independent Environment

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The codebase coordinates multi-agent configurations, user settings management, and file persistence across heterogeneous host environments.
- Dispersed components throughout the application require consistent resolution of system paths, global configuration directories, and environment variable overrides.
- Direct invocation of low-level runtime filesystem bindings across disparate modules causes divergent error handling and inconsistent directory resolution behaviors.
- The codebase establishes a shared internal utility module to encapsulate filesystem interactions, configuration directory discovery, and uniform error logging.

## Problem Statement

Uncoordinated direct interactions with runtime filesystem primitives lead to inconsistent handling of environment variables, diverging path resolution strategies, and fragmented error reporting across agent integrations, settings handlers, and configuration management modules.

## Decision

1. MUST_NOT: Domain modules MUST NOT implement independent environment variable inspection routines for configuration directory resolution when equivalent capabilities exist within FileSystemUtils.

## Policy Block

- MUST_NOT Domain modules MUST NOT implement independent environment variable inspection routines for configuration directory resolution when equivalent capabilities exist within FileSystemUtils.

In scope:
- All internal services, command handlers, agent adapters, and configuration loaders performing filesystem access.
- Directory resolution routines inspecting operating system environment variables for configuration paths.

Out of scope:
- In-memory data transformations that do not interact with persistent storage or filesystem paths.
- External third-party libraries that manage isolated storage engines independently.

Exceptions:
- EXC-20-001: Low-level bootstrap routines execute prior to the availability of the shared utility module.

## Rationale

- Static analysis demonstrates widespread adoption of FileSystemUtils across settings handlers, path managers, agent adapters, and lifecycle utilities.
- Encapsulating environment variable lookups and platform-specific path structures within a single module ensures uniform configuration discovery across differing environments.
- Centralizing error reporting and filesystem interactions prevents divergent diagnostic logging across functional boundaries.

## Consequences

Positive:
- Standardizes filesystem interaction semantics, error handling, and diagnostic logging across all consuming subsystems.
- Isolates platform-specific environment variable lookups and directory resolution logic to a single maintainable boundary.
- Eliminates duplicate path resolution and error-handling boilerplate across agent adapters, settings managers, and CLI utilities.

Negative:
- Creates an internal coupling where changes to the shared utility interface can impact multiple consuming subsystems.
- Requires contributors to adhere to internal module conventions rather than invoking standard platform filesystem APIs directly.

## Alternatives

- Direct invocation of runtime filesystem primitives within each consuming module (rejected)
  Rejected because: Results in duplicated error handling, inconsistent environment variable resolution, and platform-specific path divergence across modules.
  When valid: Standalone single-file scripts with no shared domain context or cross-platform requirements.
- Adoption of an external virtual filesystem abstraction layer (rejected)
  Rejected because: Introduces unnecessary external dependency overhead and indirection for straightforward local filesystem operations.
  When valid: Applications requiring multi-cloud pluggable storage abstractions alongside local disk access.

## Risks

- Breaking changes in the centralized utility interface can ripple across multiple consuming subsystems.
  Mitigation: Maintain strict backward-compatible interface contracts and comprehensive test coverage across all utility functions.
  Owner: Core Engineering Team
- Developers might bypass the shared module and directly invoke runtime filesystem APIs.
  Mitigation: Configure static analysis linting rules to disallow unapproved low-level filesystem imports across domain packages.
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
- Consuming modules should import functional capabilities directly from the centralized utility rather than duplicating path manipulation and inspection logic.
- All environment variable lookups for user configuration directories must follow the standardized fallback chain implemented within the core utility.

## Continuation Context


Verify commands:
- Discover the project static analysis and linting scripts from project configuration and execute them to verify import boundaries.
- Discover the project test runner through repository configuration and execute the full test suite covering filesystem utility operations.

Accept when:
- All unit and integration tests for the centralized filesystem utility and consuming modules pass without errors.
- Static analysis confirms zero unauthorized direct runtime filesystem imports across domain packages.
- Verification confirms that components across agent, settings, path resolution, and configuration domains route operations through the centralized module.

## Enforcement

- Verified by: Automated continuous integration workflows executing static analysis and linting checks.
- Verified by: Peer code reviews verifying that filesystem interactions route through the centralized utility module.
- Violation handling: Continuous integration checks will fail on unapproved direct imports of low-level filesystem primitives in domain code.
- Violation handling: Pull requests violating module encapsulation must be refactored to consume the centralized utility prior to merge approval.
- Exception process: Submit an exception request detailing the low-level architectural constraint preventing the use of the centralized module.
- Exception process: Obtain formal approval from the architecture team before committing bypassing implementations.
- Exception process: Document approved exceptions with a link to the recorded architecture review decision.