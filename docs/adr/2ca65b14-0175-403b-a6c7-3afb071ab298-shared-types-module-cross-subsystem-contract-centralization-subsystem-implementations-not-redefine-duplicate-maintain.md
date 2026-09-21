# Shared Types Module: Cross-Subsystem Contract Centralization: Subsystem Implementations Not Redefine Duplicate Maintain

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The codebase coordinates agent definitions and protocol propagation adapters across separate functional boundaries.
- Subsystems require unified data structures and interface definitions to interchange messages, server configurations, and agent capabilities without tight circular coupling.
- Static analysis identifies a shared internal types module adopted across agent interfaces and protocol merging modules.
- Maintaining decentralized or duplicate interface definitions across adapter modules introduces schema drift and maintenance overhead.

## Problem Statement

When multiple subsystem modules define or consume overlapping interface contracts, decentralized or ad-hoc type declarations cause structural drift, contract incompatibility, and tight coupling between consumer implementations. The architecture requires a unified, authoritative internal module to host shared type definitions, ensuring consistent cross-boundary contracts while isolating interface specifications from concrete business logic.

## Decision

1. MUST_NOT: Subsystem implementations MUST NOT redefine, duplicate, or maintain shadow copies of interfaces and data structures established within the shared types module.

## Policy Block

- MUST_NOT Subsystem implementations MUST NOT redefine, duplicate, or maintain shadow copies of interfaces and data structures established within the shared types module.

In scope:
- Interface definitions, domain contracts, and data transfer schemas shared across distinct subsystem boundaries.
- Agent abstractions and protocol propagation modules interacting with common data models.

Out of scope:
- Private helper types and internal implementation details strictly scoped to a single file or subsystem.
- External third-party module declarations governed by upstream vendor specifications.

Exceptions:
- EXC-20-001: A subsystem requires rapid prototyping of an unreleased protocol extension prior to contract standardization.

## Rationale

- Centralizing domain interfaces within a dedicated shared types module prevents interface divergence and inconsistent contract definitions between agent abstractions and protocol adapters.
- Enforcing a centralized type module establishes a single source of truth for message shapes and server definitions, simplifying cross-boundary refactoring.
- Decoupling type declarations from concrete implementations prevents circular dependencies between agent modules and protocol propagation mechanisms.

## Consequences

Positive:
- Guarantees contract uniformity across agent interfaces and protocol synchronization modules.
- Eliminates redundant type definitions and mitigates schema drift across adapters.
- Simplifies maintenance by providing an isolated location for shared data structures.

Negative:
- Modifications to shared type definitions can introduce breaking changes across multiple consuming subsystems simultaneously.
- Requires strict code review discipline to prevent domain-agnostic bloat within the shared types module.

## Alternatives

- Decentralized Subsystem Type Definitions (rejected)
  Rejected because: Duplicating interface definitions across agent and protocol modules creates type drift, reduces type safety, and complicates contract updates across subsystem boundaries.
  When valid: Valid only in completely decoupled microservices where subsystems do not share a common codebase or runtime environment.
- Colocating Types Within Concrete Subsystem Modules (rejected)
  Rejected because: Importing types from concrete implementation modules causes circular dependencies and entangles interface definitions with runtime behavior.
  When valid: Valid only for internal utility types used exclusively within that specific subsystem.

## Risks

- The shared types module accumulates low-cohesion types and evolves into an unbounded repository for unrelated data models.
  Mitigation: Enforce strict scope review criteria during pull request validation to ensure only cross-boundary interfaces are accepted.
  Owner: Architecture and Subsystem Maintainers
- Breaking changes to shared interface definitions cause widespread compile-time failures across consumers.
  Mitigation: Apply non-breaking additive changes where possible and deprecate obsolete fields across release cycles.
  Owner: Subsystem Maintainers

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Discover the relative path to the shared types module from the consumer module directory and establish direct relative module imports.
- Ensure shared type definitions remain strictly declarative, containing interfaces, type aliases, and contract signatures without runtime logic.
- Group related domain contracts into coherent exported namespaces or sub-paths within the shared types module to preserve readability.

## Continuation Context


Verify commands:
- Discover and execute the project type-checking script to confirm all shared interface references resolve without type errors.
- Discover and run the project static analysis and linting verification suite to validate module boundary constraints and import compliance.
- Discover and execute the repository unit and integration test suite to verify subsystem contract compatibility.

Accept when:
- Type checking completes with zero diagnostic errors across all consumer subsystems.
- Static analysis validates that no cross-subsystem type duplication or circular module dependencies exist.
- All subsystem unit and integration tests pass verifying consistent contract serialization and protocol propagation.

## Enforcement

- Verified by: Automated continuous integration pipeline running type verification and lint analysis on every pull request.
- Verified by: Architectural peer review verifying that new cross-subsystem contracts reside in the shared types module.
- Violation handling: Pull requests containing duplicate cross-subsystem interface definitions or circular type imports are blocked from merging.
- Violation handling: Violating code must be refactored to consume the centralized shared types module before re-review.
- Exception process: Submit an architectural review request with written justification explaining why the type cannot be hosted centrally, approved by two maintainers.