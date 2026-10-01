# AbstractAgent Base Module Adoption: Agent Modules Extending Abstractagent Implement Common

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Heterogeneous agent implementations require uniform execution lifecycles, configuration discovery, and common system operations across diverse AI assistant integrations.
- Direct ad-hoc implementation of individual agent classes leads to fragmented interfaces and inconsistent integration boundaries across the agent subsystem.
- Centralizing common agent behavior within an internal core module provides structural consistency and enforces contract conformance across all agent modules.

## Problem Statement

Multiple specialized agent integrations require shared lifecycle coordination, filesystem interactions, and consistent interface contracts, which creates risk of behavioral divergence and duplicated utility logic if implemented independently without a common base abstraction.

## Decision

1. MUST: All agent modules extending AbstractAgent MUST implement the common agent interface contract exposed by the agent subsystem.

## Policy Block

- MUST All agent modules extending AbstractAgent MUST implement the common agent interface contract exposed by the agent subsystem.

In scope:
- Authoring new agent integrations and concrete assistant implementations.
- Refactoring existing agent modules within the agent subsystem.

Out of scope:
- Standalone utility modules that do not represent agent implementations or assistant integrations.
- Third-party client adapters that do not participate in the core agent execution lifecycle.

## Rationale

- Standardizing on the AbstractAgent base module guarantees consistent lifecycle handling and reduces duplicated boilerplate across all agent integrations.
- Subclassing AbstractAgent establishes predictable contracts across the codebase, simplifying testing and subsystem registration.
- Codebase evidence across multiple agent modules demonstrates established convergence on the AbstractAgent abstraction for core integration patterns.

## Consequences

Positive:
- Eliminates redundant agent lifecycle management code across specialized agent integrations.
- Guarantees consistent contract compliance and structural uniformity across the agent subsystem.
- Simplifies introduction of subsequent agent implementations through inherited baseline functionality.

Negative:
- Introduces class inheritance coupling where changes to the base module cascade to all derived agent implementations.
- Requires new agent integrations to align with the base module lifecycle even when specialized behaviors diverge.

## Alternatives

- Direct interface implementation without an abstract base module (rejected)
  Rejected because: Reimplementing interface contracts independently duplicates lifecycle handling and filesystem interaction code across all agent modules.
  When valid: Valid when agent architectures share no common operational mechanics or utility dependencies.
- Functional composition pipeline instead of class-based base inheritance (deferred)
  Rejected because: Existing agent implementations demonstrate an established object-oriented class hierarchy centered on AbstractAgent.
  When valid: Valid if the subsystem undergoes a complete architectural transition toward pure functional workflows.

## Risks

- Base module bloat caused by accumulating provider-specific edge-case logic within AbstractAgent.
  Mitigation: Restrict AbstractAgent to universal lifecycle mechanics and delegate provider-specific handling to derived implementations.
  Owner: engineering team
- Fragile base class modifications breaking downstream derived agents.
  Mitigation: Enforce regression test suites covering all concrete derived agent implementations prior to base class modifications.
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
- Subclasses extending AbstractAgent must invoke base constructor methods and satisfy required abstract lifecycle hooks.
- Expose concrete implementations via the subsystem index module to maintain clean module boundaries for external consumers.

## Continuation Context


Verify commands:
- Discover the project test execution script from repository manifests and run all unit tests targeting the agent subsystem.
- Discover the static analysis and type checking verification scripts from repository manifests and run them across all agent modules.

Accept when:
- All concrete agent modules compile successfully and inherit from the AbstractAgent base module.
- Project test suites and static analysis verification pass with zero errors across all agent implementations.

## Enforcement

- Verified by: Automated static analysis and type verification executed during continuous integration.
- Verified by: Mandatory architectural peer review of pull requests introducing or modifying agent modules.
- Violation handling: Pull requests containing agent implementations that bypass the AbstractAgent base module will be blocked until refactored.
- Violation handling: Code review rejections requiring alignment with the shared agent base architecture.
- Exception process: Submit an architectural review request with technical justification for bypassing the base module to the engineering team.