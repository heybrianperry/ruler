# FileSystemUtils Internal Module for Centralized Filesystem Operations: Agent Adapters Path Resolvers Protocol Propagation

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Internal agent adapters and protocol synchronization routines require repeated filesystem interactions, including path resolution, file reads, and structured data serialization.
- Seven distinct modules across agent definitions and configuration handlers import FileSystemUtils alongside filesystem and parsing capabilities.
- Uncoordinated direct filesystem calls across disparate modules risk fragmented error handling, inconsistent path normalization, and duplicated file verification logic.
- Establishing an internal utility module consolidates common filesystem behaviors into a single shared abstraction.

## Problem Statement

Agent integrations and configuration propagation modules require dependable file access and path resolution routines. Performing unconstrained, ad-hoc filesystem operations across distributed modules leads to inconsistent file verification, divergent error handling during file read or write failures, and code duplication across agent implementations. A standardized internal module is required to centralize common filesystem operations and encapsulate path safety guarantees.

## Decision

1. MUST: All agent adapters, path resolvers, and protocol propagation modules MUST route shared filesystem interactions and path normalization through the internal FileSystemUtils module rather than reimplementing ad-hoc disk operations.

## Policy Block

- MUST All agent adapters, path resolvers, and protocol propagation modules MUST route shared filesystem interactions and path normalization through the internal FileSystemUtils module rather than reimplementing ad-hoc disk operations.

In scope:
- Components implementing agent adapters, path helpers, and configuration propagation handlers that require filesystem interaction.
- Modules performing configuration file inspection, structured document parsing, and directory resolution.

Out of scope:
- Pure utility modules and type definition files that do not perform input/output operations.
- In-memory transformation pipelines that operate strictly on received payload buffers without disk persistence.

Exceptions:
- EXC-20-001: A module requires specialized low-level streaming or OS-specific descriptors not provided by the utility module.

## Rationale

- Evidence across multiple agent and protocol modules confirms FileSystemUtils as the adopted standard for filesystem abstraction.
- Centralizing filesystem logic prevents fragmentation of path resolution logic and unifies operational safeguards across diverse agent integrations.
- Encapsulating filesystem mechanics within a shared module minimizes the maintenance overhead when underlying runtime file handling contracts evolve.

## Consequences

Positive:
- Eliminates duplicate filesystem and path management routines across agent and propagation components.
- Standardizes error resilience and file handling behaviors across multiple distinct agent implementations.
- Simplifies auditing and testing of filesystem boundaries by consolidating disk interactions into an isolated internal module.

Negative:
- Couples consuming modules to the shared interface and lifecycle of the internal utility module.
- Requires ongoing maintenance of the shared module to support varied access patterns across disparate agent designs.

## Alternatives

- Permit each agent and protocol module to manage independent direct filesystem calls using core runtime libraries. (rejected)
  Rejected because: Leads to fragmented error handling, duplicated path resolution logic, and inconsistent file verification across integrations.
  When valid: Valid only in standalone single-file scripts with no shared architectural dependencies.
- Adopt an external third-party virtual filesystem abstraction library. (rejected)
  Rejected because: Introduces unnecessary external dependency overhead when an internal utility module already satisfies project isolation requirements.
  When valid: Valid when multi-cloud virtual storage backends or distributed filesystem mocking are required.

## Risks

- Breaking changes to the FileSystemUtils public interface may propagate regressions to multiple dependent agent modules.
  Mitigation: Maintain strict backward compatibility and semantic interface testing for the shared utility module.
  Owner: Core Engineering Team
- Uncaught exceptions from filesystem operations within the utility may degrade agent resilience.
  Mitigation: Enforce defensive exception handling, fallback defaults, and non-fatal warning logging across all utility methods.
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
- Verify that any new filesystem utility methods expose deterministic, promise-based or synchronous interfaces consistent with existing consumer patterns.
- Ensure all configuration parsing operations coupled with filesystem retrieval include validation layers to guard against malformed data inputs.

## Continuation Context


Verify commands:
- Discover and execute project verification test suites covering filesystem utility functions and consuming agent adapters.
- Run repository static analysis and linting checks to confirm module imports adhere to internal architectural boundaries.

Accept when:
- All unit and integration tests exercising FileSystemUtils and dependent agent adapters complete with zero errors.
- Static boundary analysis confirms that all filesystem interactions within agent and propagation modules route through approved internal utilities.

## Enforcement

- Verified by: Automated continuous integration test and static analysis pipelines.
- Verified by: Peer code review for changes involving filesystem access or agent implementations.
- Violation handling: Pull requests introducing direct unapproved filesystem calls or bypassing the utility module will be blocked until refactored.
- Exception process: Submit an architectural exception request detailing why the internal utility module is insufficient, subject to approval by the architecture review team.