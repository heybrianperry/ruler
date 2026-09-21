# IAgent Core Agent Interface Module Adoption: Engineers Integrating External Dependencies Updating Runtime

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The project coordinates agent tasks and rollback capabilities across heterogeneous directory trees governed by configuration entries.
- Multiple core execution modules, including primary application routines and revert workflows, require access to resolved agent instances.
- Static analysis identifies consistent imports of the IAgent contract and directory-keyed agent caching across execution and revert pipelines.

## Problem Statement

Orchestration workflows across forward execution and revert phases require a consistent mechanism to resolve, invoke, and track agent behaviors without coupling core pipelines to concrete agent implementations or repeatedly recomputing directory-scoped configurations.

## Decision

1. MUST: Engineers integrating external dependencies or updating runtime libraries MUST discover the project dependency manifest and lock file to resolve and pin the exact dependency versions.

## Policy Block

- MUST Engineers integrating external dependencies or updating runtime libraries MUST discover the project dependency manifest and lock file to resolve and pin the exact dependency versions.

In scope:
- Orchestration modules executing agent tasks or managing agent lifecycles across directory configurations
- Rollback and revert routines coordinating agent-specific reversal operations

Out of scope:
- Low-level system utility functions that do not execute or inspect agent behaviors
- Independent test fixtures mocking raw file system operations without agent dispatch

## Rationale

- Import declarations across both forward execution and revert modules substantiate shared reliance on the IAgent interface contract to ensure behavioral consistency.
- Caching resolved agents by configuration directory avoids redundant agent initialization across nested directory hierarchies.
- Standardizing on a unified internal interface enables symmetric execution and revert workflows without duplicating concrete agent dispatch logic.

## Consequences

Positive:
- Guarantees behavioral symmetry between forward execution and reversal workflows through a shared agent contract.
- Decouples orchestration modules from concrete agent class implementations, facilitating extension of new agent types.
- Minimizes configuration re-parsing and duplicate agent instantiations across directory hierarchies.

Negative:
- Changes to the IAgent contract require coordinated updates across all implementing agents and consuming orchestrators.
- Directory-based agent caching introduces runtime state management overhead across orchestration invocations.

## Alternatives

- Direct concrete agent class instantiation within each orchestration module (rejected)
  Rejected because: Couples orchestration workflows directly to concrete agent implementations and breaks symmetric rollback handling.
  When valid: Single-agent scripts with no dynamic agent selection or rollback requirements.
- Dynamic untyped agent invocation via generic configuration dictionaries (rejected)
  Rejected because: Eliminates compile-time interface verification and increases runtime type failure risks during execution and reversion.
  When valid: Ad-hoc plugin architectures with dynamic external script loading.

## Risks

- Interface drift between execution and revert lifecycles if agent capabilities diverge
  Mitigation: Enforce strict compile-time interface compliance and unified regression test suites spanning both execution and reversal.
  Owner: engineering team
- Cache invalidation issues if configuration directories mutate during a single execution run
  Mitigation: Treat configuration directory mappings as immutable records scoped to the lifecycle of the execution command.
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
- Consumers must inspect the repository to locate the internal agent interface definitions and implement all required interface methods when introducing new agent capabilities.
- Consumers must verify agent resolution mechanisms preserve directory-keyed scoping across both execution and revert workflows.

## Continuation Context


Verify commands:
- Discover the project repository test script from the package configuration and execute the complete test suite.
- Discover the repository type-checking configuration and execute compile-time interface verification.

Accept when:
- All unit and integration tests covering agent execution and reversal pass without failures.
- Static type analysis confirms complete compliance with the agent interface contract without type errors.

## Enforcement

- Verified by: Automated continuous integration checks executing repository test and type-check scripts
- Verified by: Peer code review verifying adherence to the shared agent interface contract and directory-keyed scoping
- Violation handling: Pull requests bypassing the agent interface contract or introducing direct concrete dependencies will be blocked.
- Violation handling: Code reviews require refactoring concrete agent coupling into interface-compliant implementations before merge.
- Exception process: Exceptions require documented architectural approval from team leads outlining why the standard agent contract cannot satisfy specific execution requirements.