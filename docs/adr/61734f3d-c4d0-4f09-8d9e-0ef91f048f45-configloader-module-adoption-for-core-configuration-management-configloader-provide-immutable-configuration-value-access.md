# ConfigLoader Module Adoption for Core Configuration Management: Configloader Provide Immutable Configuration Value Access

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Core operational subsystems require consistent access to runtime and operational configuration parameters across multiple engine components.
- Direct file system access and ad-hoc configuration resolution within individual domain modules lead to duplicated parsing logic and configuration drift.
- The codebase establishes a centralized internal module pattern where core components delegate configuration retrieval and parsing to ConfigLoader.

## Problem Statement

Without a unified configuration access mechanism, individual core operational modules independently parse environment variables or storage artifacts, leading to inconsistent fallback handling, tight coupling to underlying storage mechanisms, and fragmented configuration validation.

## Decision

1. SHOULD: ConfigLoader SHOULD provide immutable configuration value access to consumer modules to prevent runtime mutations.

## Policy Block

- SHOULD ConfigLoader SHOULD provide immutable configuration value access to consumer modules to prevent runtime mutations.

In scope:
- Core operational components and engine subsystems requiring runtime or operational configuration.
- Module boundaries interfacing with system-level configuration parameters.

Out of scope:
- Isolated utility functions that operate strictly on explicit parameters passed by callers without configuration dependencies.
- Ephemeral test fixtures utilizing mock configuration objects directly in test execution environments.

## Rationale

- Delegating configuration management to ConfigLoader centralizes parameter parsing and schema validation, preventing divergent configuration states across core operational modules.
- Decoupling operational routines such as engine reversion and agent selection from configuration storage mechanisms improves testability and modularity.
- Evidence across multiple core modules confirms an existing dependency on ConfigLoader, establishing it as the standard configuration interface for core operations.

## Consequences

Positive:
- Standardizes configuration access across core operational engines, ensuring consistent resolution behavior.
- Simplifies unit and integration testing by allowing test suites to mock a single configuration provider.
- Eliminates duplicate parsing and environment resolution logic from domain-specific modules.

Negative:
- Introduces a shared dependency across core modules that requires disciplined interface stability.
- Requires new core features to adhere to ConfigLoader schema definitions rather than reading environment properties directly.

## Alternatives

- Direct Environment and File System Access in Core Modules (rejected)
  Rejected because: Causes duplicated parsing logic, inconsistent default handling, and tight coupling between operational domain logic and storage details.
  When valid: Lightweight single-purpose scripts that have no shared core module dependencies.
- Global Configuration Singleton Access (rejected)
  Rejected because: Obscures module dependency graphs and complicates testing by introducing hidden shared mutable state.
  When valid: Trivially simple command-line utilities without complex modular orchestration requirements.

## Risks

- Changes to ConfigLoader interface could break multiple dependent core modules simultaneously.
  Mitigation: Enforce strict interface contracts and comprehensive regression test suites for all configuration access methods.
  Owner: Core Subsystem Team
- Consumer modules might circumvent ConfigLoader during urgent refactoring.
  Mitigation: Implement static analysis checks and pull request review gates that prohibit direct environment or disk reads in core modules.
  Owner: Engineering Architecture Team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Core modules must inject or import ConfigLoader using the repository module resolution conventions.
- Configuration keys must be defined within structured configuration contracts exported by the configuration module.

## Continuation Context


Verify commands:
- Discover the project static analysis and linting scripts from the repository build manifest and execute them across core subsystem source modules.
- Discover the test runner command from the repository build manifest and execute all unit and integration test suites covering core modules.

Accept when:
- Static analysis confirms zero direct environment or file system configuration reads within core operational modules outside ConfigLoader.
- All unit and integration tests covering core modules pass with configuration provided through ConfigLoader.

## Enforcement

- Verified by: Automated static analysis rules preventing direct environment variable access in core modules.
- Verified by: Mandatory peer review on pull requests touching core subsystem modules.
- Violation handling: Pull requests containing direct environment reads or file configuration parsing outside ConfigLoader are blocked by continuous integration.
- Violation handling: Violating implementations must be refactored to consume configuration via ConfigLoader before approval.
- Exception process: Exceptions require approval from the architecture team documented in an architectural review issue describing why ConfigLoader cannot meet the specific need.