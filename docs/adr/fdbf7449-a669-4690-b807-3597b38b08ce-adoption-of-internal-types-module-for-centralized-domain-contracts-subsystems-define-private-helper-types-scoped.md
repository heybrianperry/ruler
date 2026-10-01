# Adoption of Internal Types Module for Centralized Domain Contracts: Subsystems Define Private Helper Types Scoped

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The project codebase coordinates multiple operational domains including editor configuration, protocol management, and execution skills.
- Multiple independent modules require consistent structural representations of shared domain entities and communication payloads.
- Static code analysis reveals uniform import of a centralized internal types module across all subsystem boundaries.

## Problem Statement

When multiple subsystems interact with shared domain entities and communication payloads, localized or fragmented type declarations lead to interface drift, inconsistent data handling, and runtime errors. A centralized, authoritative module contract is necessary to guarantee structural consistency and compile-time type safety across all system boundaries.

## Decision

1. MAY: Subsystems MAY define private helper types scoped strictly to internal module implementation details if those types never cross module boundaries.

## Policy Block

- MAY Subsystems MAY define private helper types scoped strictly to internal module implementation details if those types never cross module boundaries.

In scope:
- Domain models, data transfer objects, configuration interfaces, and protocol definitions shared across subsystems.
- Modules in editor integrations, protocol adapters, and core execution layers exchanging structured entities.

Out of scope:
- Implementation-private local variables, internal iteration counters, and unexported helper types confined to a single file.
- Self-contained third-party library wrappers that do not expose types across subsystem boundaries.

## Rationale

- Adopting a shared internal types module ensures that changes to data contracts propagate immediately at compile time to all consuming subsystems.
- Evidence shows zero reliance on fragmented local interfaces for cross-boundary data transfer across editor, protocol, and core components.
- Consolidating domain types minimizes maintenance burden and prevents regressions in protocol encoding and configuration parsing.

## Consequences

Positive:
- Guarantees compile-time consistency of domain models and payload structures across all subsystems.
- Eliminates redundant type definitions and reduces maintenance overhead when data models evolve.
- Provides a single source of truth for interfaces consumed across integration layers.

Negative:
- Couples disparate subsystem modules to changes within the shared contract module.
- Requires cross-subsystem coordination when evolving shared definitions.

## Alternatives

- Decentralized Subsystem Type Definitions (rejected)
  Rejected because: Duplicating entity definitions in each subsystem causes structural drift and requires redundant maintenance across components.
  When valid: When subsystems have completely disjoint operational domains with zero shared data contracts.
- External Schema Registry or Serialization IDL (deferred)
  Rejected because: Introduces external tooling and generation overhead that is unnecessary for a single-language internal codebase.
  When valid: When crossing polyglot language boundaries or deploying heterogeneous microservices.

## Risks

- Uncoordinated modifications to shared contracts may introduce breaking changes across dependent subsystems.
  Mitigation: Enforce contract review and automated type checking across all workspaces during continuous integration.
  Owner: engineering team
- The shared module may accumulate unrelated or overly specific definitions over time.
  Mitigation: Enforce strict domain scoping during review to keep definitions focused on cross-boundary contracts.
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
- When modifying shared data contracts, inspect all dependent module interfaces across subsystems to ensure contract compatibility.
- Group cohesive domain entity interfaces within distinct exports inside the shared module to facilitate clear dependency boundaries.

## Continuation Context


Verify commands:
- Discover the workspace static type checker from project manifests and execute the project-wide type validation script.
- Discover and run the project automated test suite to ensure contract compatibility.

Accept when:
- All subsystem modules import shared contracts from the designated internal types module without local interface duplication.
- Static type analysis across the entire project repository completes with zero errors.

## Enforcement

- Verified by: Automated static type analysis executed in continuous integration pipelines.
- Verified by: Peer code review verifying adherence to centralized contract imports.
- Violation handling: Pull requests containing duplicate cross-boundary type definitions or failing type checks must be blocked until resolved.
- Exception process: Exceptions require documented architectural justification and sign-off from the technical lead explaining why a domain contract cannot reside in the shared module.