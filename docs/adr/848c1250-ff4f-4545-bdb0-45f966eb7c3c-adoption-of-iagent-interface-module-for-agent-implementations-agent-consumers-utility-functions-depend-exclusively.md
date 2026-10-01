# Adoption of IAgent Interface Module for Agent Implementations: Agent Consumers Utility Functions Depend Exclusively

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Diverse coding assistant and automation agents are integrated across the agent subsystem, requiring a uniform mechanism for invocation, lifecycle control, and capability declaration.
- Without a shared contract, individual agent integrations diverge in lifecycle management, interaction protocols, and parameter handling.
- The internal IAgent interface module is imported across agent implementations and agent utilities to enforce structural uniformity.

## Problem Statement

Integrating multiple diverse coding assistant integrations without an explicit structural boundary creates tight coupling, inconsistent lifecycle execution, and fragmented dispatch logic across agent consumers.

## Decision

1. MUST: Agent consumers and utility functions MUST depend exclusively upon the IAgent contract rather than concrete agent implementation classes when referencing, orchestrating, or dispatching agents.

## Policy Block

- MUST Agent consumers and utility functions MUST depend exclusively upon the IAgent contract rather than concrete agent implementation classes when referencing, orchestrating, or dispatching agents.

In scope:
- Agent integration modules, agent utility routines, and agent orchestration dispatchers within the agent subsystem.
- New coding assistant or automation agent integrations being introduced to the codebase.

Out of scope:
- Non-agent utility modules and foundational file system utilities that do not participate in agent execution.
- External third-party provider libraries wrapped behind agent adapter boundaries.

## Rationale

- Static analysis confirms that the IAgent module is adopted uniformly across all seven agent-related files in the subsystem, establishing a cohesive polymorphic abstraction boundary.
- Centralizing integration contracts around IAgent decouples high-level orchestration workflows from provider-specific protocols and runtime behaviors.
- Encapsulating agent operations behind a common contract ensures uniform error handling, lifecycle observability, and modular testability.

## Consequences

Positive:
- Enables polymorphic interchangeability and orchestration of disparate agent providers across consumer modules.
- Guarantees a consistent operational contract for lifecycle execution, messaging, and storage interaction across agent modules.
- Simplifies unit test verification by isolating test doubles behind a single interface boundary.

Negative:
- Requires adapter layers and structural compromises when integrating provider-specific capabilities that fall outside the common IAgent interface.
- Modifications to the shared IAgent interface require coordinated updates across all implementing agent classes.

## Alternatives

- Direct consumption of individual external provider APIs without an internal interface module (rejected)
  Rejected because: Couples agent orchestration logic directly to external provider APIs, leading to fragmented error handling and architectural drift across callers.
  When valid: When only a single immutable agent backend is supported across the lifetime of the application.
- Dynamic duck typing and runtime inspection without static interface contracts (rejected)
  Rejected because: Removes compile-time type verification, increasing the likelihood of unhandled execution paths and runtime interface mismatches.
  When valid: In dynamically interpreted environments where static interface contracts are unsupported.

## Risks

- Provider-specific capabilities may be constrained or awkward to expose through a generalized interface contract.
  Mitigation: Design extensible capability negotiation or metadata payload hooks within the interface definition.
  Owner: engineering team
- Interface evolution breaking backward compatibility across multiple existing agent implementations.
  Mitigation: Enforce strict semantic versioning and default method implementations within the shared abstract base class.
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
- Ensure that all newly introduced agent providers provide concrete implementations satisfying the IAgent contract before exposing them in the agent index export.
- Shared capabilities and foundational file system interactions should be centralized in an abstract base agent class rather than duplicated across individual agent implementations.

## Continuation Context


Verify commands:
- Discover the repository typecheck or build script from the project manifest and execute it to verify complete interface compliance across all agent implementations.
- Discover the repository test runner script and execute the agent test suite to ensure polymorphic execution across all agent implementations.

Accept when:
- All agent integration classes strictly implement the IAgent contract with zero type checking or compilation errors.
- Agent orchestration workflows invoke diverse agent implementations solely through the IAgent contract.

## Enforcement

- Verified by: Automated continuous integration build checks verifying interface conformance and static typing.
- Verified by: Architecture and peer code review for all pull requests modifying or adding agent implementations.
- Violation handling: Continuous integration failures prevent merging code containing agent implementations that do not satisfy the IAgent contract.
- Violation handling: Code reviews reject direct consumer imports of concrete agent implementation classes in orchestration modules.
- Exception process: Exceptions must be submitted via an architecture review RFC detailing why the provider cannot conform to the IAgent contract, approved by the lead architect.