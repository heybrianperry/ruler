# Adoption of FileSystemUtils Core Module: Modules Not Invoke Raw Low Level

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Multiple agent integration adapters and configuration settings services require disk access to inspect workspaces, read specifications, and persist settings.
- Standard platform filesystem operations require repetitive path resolution, safety validation, and error management across components.
- A shared core module, FileSystemUtils, has been adopted across agent implementations and configuration services to standardize filesystem interactions.

## Problem Statement

Decentralized filesystem interactions across various agent adapters and configuration modules risk inconsistent file handling, redundant path normalization logic, and disparate error recovery mechanisms. The project requires a standardized approach to manage filesystem utilities and operations through a unified core module.

## Decision

1. MUST_NOT: Modules MUST_NOT invoke raw low-level filesystem methods when an equivalent standardized helper is provided by FileSystemUtils.

## Policy Block

- MUST_NOT Modules MUST_NOT invoke raw low-level filesystem methods when an equivalent standardized helper is provided by FileSystemUtils.

In scope:
- Agent adapter modules requiring filesystem interactions and workspace inspections.
- Configuration management components that read, parse, or persist local settings.

Out of scope:
- Pure in-memory domain logic and stateless protocol message transformers.
- Third-party platform integrations that do not interact with local file systems.

## Rationale

- Consolidating common filesystem operations into FileSystemUtils prevents duplicate code across agent adapters and simplifies adapter maintenance.
- Centralized utilities ensure consistent path handling and file manipulation semantics across heterogeneous agent integrations.
- Evidence demonstrates concurrent adoption across multiple agent modules and configuration components, reflecting an established architectural convention.

## Consequences

Positive:
- Provides a single authoritative module for shared filesystem routines across all agent implementations.
- Reduces boilerplate code related to path manipulation and file handling in individual agent adapters.
- Improves maintainability by isolating filesystem modifications to a centralized utility surface.

Negative:
- Couples multiple independent agent adapters to the internal FileSystemUtils module API surface.
- Requires ongoing maintenance of shared utility abstractions alongside platform filesystem updates.

## Alternatives

- Direct unabstracted reliance on platform filesystem and path modules across all adapters (rejected)
  Rejected because: Duplicating filesystem logic across every agent adapter leads to inconsistent path resolution, redundant error handling, and maintenance drift.
  When valid: Valid in minimal standalone scripts with single-file scope where shared abstractions introduce unnecessary overhead.
- Complete dependency injection of a virtualized filesystem interface into each adapter (deferred)
  Rejected because: Full filesystem virtualization introduces additional architectural complexity not currently required by the codebase.
  When valid: Valid when multi-tenant cloud sandboxing or in-memory virtual filesystems are required for test isolation.

## Risks

- Broad adoption of a single utility module creates a central coupling point where breaking changes impact all agent adapters.
  Mitigation: Enforce strict backwards-compatible method signatures and comprehensive unit testing for all exported utilities.
  Owner: Core Architecture Team
- Leaking platform-specific path assumptions through the utility layer causing cross-platform failures.
  Mitigation: Standardize on platform-agnostic path normalization functions within the utility module.
  Owner: Core Architecture Team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When adding new filesystem capabilities needed across multiple agents, extend FileSystemUtils rather than embedding custom routines inside individual adapters.
- Ensure all filesystem operations exported by FileSystemUtils handle asynchronous exceptions and normalize path separators consistently across operating environments.

## Continuation Context


Verify commands:
- Discover and execute the project verification script from the repository manifest to validate module import compliance.
- Execute the repository test runner discovered through workspace configuration to confirm filesystem utility integration tests pass.

Accept when:
- All agent adapter modules and configuration handlers access shared filesystem operations exclusively through the FileSystemUtils core module.
- All automated integration and unit test suites defined in the repository pass without module resolution or filesystem access errors.

## Enforcement

- Verified by: Automated static analysis checks verifying import boundaries across agent modules.
- Verified by: Peer code reviews confirming new agent adapters consume FileSystemUtils for common disk operations.
- Violation handling: Pull requests introducing duplicated filesystem helper functions or bypassing FileSystemUtils for standardized operations will be blocked during code review.
- Violation handling: Violations identified during static analysis must be refactored to use the centralized utility module.
- Exception process: Exceptions for direct low-level filesystem interactions require documented justification explaining why FileSystemUtils cannot support the operation, approved by the architecture lead.