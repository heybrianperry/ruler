# Adoption of Internal Constants Module: Consumers Discover Project Dependency Manifest Lock

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Core domain processors, agent selection routines, and processing utilities require access to shared configuration values and domain parameters across the subsystem.
- Multiple core execution components independently consume cross-cutting constants to coordinate agent selection and processing workflows.
- Decentralized or duplicated value definitions create configuration divergence and maintainability risks across interconnected processing components.
- The codebase establishes a centralized internal constants module to provide a single authoritative source for shared domain values.

## Problem Statement

Core processing modules and agent selection components require shared domain values, operational thresholds, and configuration constants. Without a standardized, centralized module for shared constants, definitions become duplicated across individual processing units, increasing the risk of configuration drift, inconsistent behavior, and maintenance overhead when operational values change.

## Decision

1. MUST: Consumers MUST discover the project dependency manifest and lock artifact to resolve all dependency versions before implementing or updating module integrations.

## Policy Block

- MUST Consumers MUST discover the project dependency manifest and lock artifact to resolve all dependency versions before implementing or updating module integrations.

In scope:
- Core execution processors, agent selection services, and shared utility modules within the core domain.
- All components that define or consume cross-cutting domain constants and shared operational parameters.

Out of scope:
- Configuration values that are strictly private to a single implementation file and never shared across boundaries.
- External environment variable parsing performed exclusively at application bootstrap.

Exceptions:
- EXC-20-001: A component requires runtime-dynamic configuration values loaded from external dynamic sources that cannot be represented as static constants.

## Rationale

- Centralizing shared values within an internal constants module establishes an authoritative single source of truth across core processors and agent selection services.
- Static analysis confirms consistent adoption of the internal constants module across four distinct core processing and selection modules, demonstrating established subsystem conventions.
- Eliminating duplicated literals reduces the risk of configuration drift and simplifies domain updates across interdependent processing modules.

## Consequences

Positive:
- Provides a single point of definition and modification for shared domain values across core processing units.
- Prevents configuration divergence and inconsistent runtime parameters across agent processors and utilities.
- Improves discoverability of available domain flags and operational constants for developers extending the core subsystem.

Negative:
- Creates a centralized dependency that core modules couple to for shared domain values.
- Requires developers to update a shared location rather than defining values inline during rapid prototyping.

## Alternatives

- Decentralized inline literal values declared locally within each processor module (rejected)
  Rejected because: Leads to magic literals, duplicate declarations, and configuration drift across interdependent core components
  When valid: When values are strictly single-use implementation details that have zero cross-cutting relevance
- Dynamic configuration loader injection for all static constants (rejected)
  Rejected because: Introduces unnecessary asynchronous initialization and runtime overhead for invariant domain definitions
  When valid: When values must change dynamically at runtime without restarting the application

## Risks

- The centralized constants module may become bloated with unrelated values from across disparate domain areas.
  Mitigation: Group constants logically by domain concern within the module and enforce review checks against adding non-core constants.
  Owner: Core Engineering Team
- Modifications to a widely imported constant could unintentionally alter behavior in dependent processors.
  Mitigation: Execute comprehensive automated regression test suites covering all dependent processors before releasing constant changes.
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
- Consumers must inspect the project module structure to identify the authoritative constants module path and import definitions using relative or aliased module specifiers matching repository conventions.
- Ensure that exported constants are declared as immutable structures to prevent downstream mutations from affecting other importing modules.

## Continuation Context


Verify commands:
- Discover and run the project static analysis and linting scripts to verify compliance with centralized constants import rules.
- Discover and execute the project unit and integration test suites to confirm that dependent processors operate correctly with centralized constant values.

Accept when:
- Static analysis checks pass with zero violations regarding unauthorized duplicated constants or magic literals in core processors.
- All automated test suites for core processors and agent selection modules pass successfully.

## Enforcement

- Verified by: Automated static analysis checks in continuous integration pipelines.
- Verified by: Peer code review for pull requests modifying core modules or shared constants.
- Violation handling: Pull requests introducing duplicate constant definitions or bypasses of the shared constants module are blocked until resolved.
- Violation handling: Violations identified post-merge are tracked as technical debt issues for refactoring.
- Exception process: Exceptions must be requested via architecture review with documented justification detailing why centralized constants cannot be utilized.