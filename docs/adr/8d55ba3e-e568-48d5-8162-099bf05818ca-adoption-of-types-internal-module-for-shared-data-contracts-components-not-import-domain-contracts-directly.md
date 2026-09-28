# Adoption of types Internal Module for Shared Data Contracts: Components Not Import Domain Contracts Directly

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Core domain logic, processors, configuration validators, and integration layers require synchronized data models and shared interfaces to exchange information reliably.
- Decentralized or ad-hoc interface declarations across distinct subsystems risk contract drift, schema duplication, and circular dependency chains.
- Static analysis reveals widespread adoption of a centralized internal types module across core utilities, processors, and configuration boundaries.

## Problem Statement

Without a centralized module for shared interfaces and data structures, independent subsystems and utility modules must either redefine domain types locally or establish cross-cutting dependencies between functional domains. This creates schema synchronization overhead, increases maintenance friction during contract evolution, and elevates the risk of circular module dependencies.

## Decision

1. MUST_NOT: Components MUST NOT import domain contracts directly from peer implementation modules across subsystem boundaries, routing all inter-domain type requirements through the centralized types module to avoid circular dependency chains.

## Policy Block

- MUST_NOT Components MUST NOT import domain contracts directly from peer implementation modules across subsystem boundaries, routing all inter-domain type requirements through the centralized types module to avoid circular dependency chains.

In scope:
- Core domain processors and utility modules handling business logic and transformations
- Configuration management and external settings adaptation layers
- Cross-subsystem communication interfaces and shared data transfer objects

Out of scope:
- Module-private helper types and internal implementation states scoped strictly to a single file
- Third-party vendor adapter types that are completely isolated within an external integration boundary

## Rationale

- Observed evidence demonstrates that multiple critical modules spanning core utilities, processing pipelines, unified configuration definitions, and settings integration uniformly depend on the centralized types module.
- Consolidating contract definitions into a dedicated module isolates type definitions from executable runtime side effects, reducing bundle coupling and preventing circular import graphs.
- A single source of truth for domain schemas ensures that changes to shared contracts are audited centrally and reflected immediately across all dependent subsystems.

## Consequences

Positive:
- Eliminates redundant type definitions and prevents structural schema drift across core and platform integration layers.
- Averts circular dependency cycles between processing utilities and configuration handlers by establishing a leaf-level contract module.
- Improves developer productivity and code navigation by centralizing domain models in a predictable location.

Negative:
- Changes to widely consumed types in the shared module can cause cascade compilation errors across multiple subsystems.
- Centralization requires disciplined governance to prevent the shared module from becoming an unmaintained dumping ground for unrelated types.

## Alternatives

- Colocating type definitions within individual feature and processing modules (rejected)
  Rejected because: Causes circular module dependencies when multiple components across distinct subsystems need to reference the same data contract
  When valid: Valid only for types strictly confined to a single self-contained component with zero external consumers
- Distributing shared contracts across multiple domain-specific type modules (deferred)
  Rejected because: Introduces unnecessary structural fragmentation and path resolution overhead for the current codebase scale
  When valid: Valid when subsystem domain boundaries become sufficiently large to justify separate workspace packages or independent domain contracts

## Risks

- Uncontrolled growth of the centralized types module leading to monolithic coupling and blurred domain boundaries
  Mitigation: Enforce domain-based sectioning and architectural code reviews for any new contract introduced into the shared module
  Owner: Core Engineering Team
- Breaking changes in shared types silently disrupting runtime behavior if serialization layers do not validate input
  Mitigation: Pair contract updates with automated schema validation tests across all external input entry points
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
- Organize shared contracts logically within the types module using clear structural categories such as domain models, configuration structures, and communication interfaces.
- Ensure that exported interfaces and types contain documentation comments describing property contracts, optionality, and domain constraints.

## Continuation Context


Verify commands:
- Discover the project static analysis and type verification script from the repository manifest and execute it to ensure zero contract compilation errors across all modules.
- Discover and execute the module dependency cycle analysis script from the build configuration to confirm no circular dependencies exist between consuming components and the types module.
- Discover and run the project automated test suite from the repository manifest to confirm runtime payload validation passes against shared contracts.

Accept when:
- Static type verification passes with zero errors across all core utilities, processors, and configuration modules.
- Dependency graph verification reports zero circular dependency cycles involving the centralized types module.
- All unit and integration test suites succeed with all assertions passing against shared contract definitions.

## Enforcement

- Verified by: Automated type checking and linting pipelines running on pull request submissions.
- Verified by: Architecture dependency boundary checks validating that peer implementation modules do not cross-import domain contracts.
- Verified by: Peer review by engineering maintainers for any modification to the centralized types module.
- Violation handling: Pull requests failing static type checks or introducing circular dependencies are automatically blocked from merging.
- Violation handling: Direct cross-subsystem type imports detected in code review must be refactored to consume the centralized types module before approval.
- Exception process: Exceptions for isolated third-party vendor adapter types must be submitted via architectural review and documented with boundary isolation justification.