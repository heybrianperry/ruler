# ConfigLoader Module Adoption: Core Engine Subsystems Not Perform Independent

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Core engine subsystems require consistent access to operational parameters and agent configuration settings.
- Direct filesystem reads and ad-hoc environment parsing across disparate core modules create configuration divergence and schema drift.
- Core engine components standardize configuration ingestion through the internal ConfigLoader module.

## Problem Statement

Without a unified configuration loading module, core engine subsystems risk implementing disparate mechanisms for reading settings, resulting in duplicate filesystem operations, inconsistent environment variable precedence, and tight coupling to underlying storage formats.

## Decision

1. MUST_NOT: Core engine subsystems MUST NOT perform independent filesystem reads or parse raw environment variables for configuration data.

## Policy Block

- MUST_NOT Core engine subsystems MUST NOT perform independent filesystem reads or parse raw environment variables for configuration data.

In scope:
- Core engine subsystem modules requiring operational parameters or runtime configuration settings.
- Subsystems coordinating agent orchestration, revert actions, and engine execution.

Out of scope:
- Standalone utility functions that operate purely on arguments passed directly at invocation.
- External system boundary adapters with isolated configuration mechanisms defined externally.

## Rationale

- Centralizing configuration ingestion in ConfigLoader ensures uniform option validation and default resolution across core subsystems.
- Decoupling core engines from direct filesystem and environment parsing minimizes redundant I/O and facilitates test isolation.
- Standardizing on a single internal configuration module ensures consistent behavior across agent selection, revert execution, and related workflows.

## Consequences

Positive:
- Establishes a single authoritative path for configuration parsing and default option resolution.
- Decouples core engine operational logic from underlying storage structures and filesystem I/O.
- Simplifies schema evolution and environment variable mapping across all dependent components.

Negative:
- Introduces an explicit internal dependency coupling core engine subsystems to ConfigLoader.
- Requires mock configuration providers or fixtures when constructing isolated tests for core consumers.

## Alternatives

- Decentralized direct environment variable and filesystem parsing within each core module (rejected)
  Rejected because: Causes duplicate parsing logic, fragmented schema handling, and inconsistent option resolution across core subsystems
  When valid: Standalone single-file scripts with no integration with wider core engine subsystems
- Unencapsulated global configuration state initialized at process startup (rejected)
  Rejected because: Obscures dependency requirements, complicates unit test isolation, and risks uncontrolled configuration mutation
  When valid: Static utility processes that do not require configurable options or test isolation

## Risks

- Failure to validate configuration options within ConfigLoader could propagate invalid state across all dependent core modules.
  Mitigation: Implement strict schema validation and structured error reporting within ConfigLoader initialization.
  Owner: Core Engineering Team
- Circular dependencies could arise if ConfigLoader imports core engine components that require configuration.
  Mitigation: Maintain a strict unidirectional dependency structure ensuring ConfigLoader acts as a leaf dependency relative to core engine consumers.
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
- Core engine modules should accept configuration parameters through constructor injection or module-level interfaces backed by ConfigLoader to facilitate unit testing.
- Configuration resolution failures within ConfigLoader must fail fast with structured diagnostic messages before core subsystem execution proceeds.

## Continuation Context


Verify commands:
- Discover the test runner defined in the project configuration and execute the core test suite to verify configuration loading behavior.
- Discover the static analysis command from the repository scripts and verify that core modules have no unauthorized configuration imports.
- Discover the build script from the repository manifest and execute a clean build to confirm interface compatibility with ConfigLoader.

Accept when:
- Core engine subsystems successfully initialize their runtime options exclusively via ConfigLoader.
- Direct parsing of filesystem configuration artifacts or raw environment variables is absent from core operational modules.
- Repository test and verification suites pass with centralized configuration resolution in place.

## Enforcement

- Verified by: Static analysis and dependency graph inspection during continuous integration to detect direct configuration reads.
- Verified by: Peer review verification for any changes touching core subsystem configuration access.
- Violation handling: Pull requests containing unauthorized direct configuration parsing or bypasses of ConfigLoader are rejected by automated checks.
- Violation handling: Code review blocks merging until configuration access is routed through ConfigLoader.
- Exception process: Exceptions must be submitted with written architectural justification and approved by core subsystem maintainers prior to code integration.