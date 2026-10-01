# Adoption of AgentsMdAgent as Base Implementation for Agent Modules: Agent Integration Modules Not Duplicate Core

Status: proposed
Date: 2025-02-18
Deciders: Detection Pipeline (automated)

## Context

- The agent subsystem contains multiple specialized agent modules that interact with external execution engines.
- Static analysis reveals repeated imports of the internal AgentsMdAgent module across agent implementations.
- A unified foundational module avoids divergent implementation details while maintaining a shared agent contract.

## Problem Statement

Multiple specialized agent integrations require consistent orchestration and lifecycle behavior, but implementing each agent independently risks duplicate execution routines, inconsistent interfaces, and high maintenance overhead.

## Decision

1. MUST_NOT: Agent integration modules MUST NOT duplicate core orchestration routines provided by AgentsMdAgent.

## Policy Block

- MUST_NOT Agent integration modules MUST NOT duplicate core orchestration routines provided by AgentsMdAgent.

In scope:
- All specialized agent adapter and integration modules within the agent subsystem.

Out of scope:
- Utility modules, core file system helpers, and presentation components outside the agent integration subsystem.

## Rationale

- Four distinct agent implementations consistently depend on AgentsMdAgent, confirming its role as the standard foundation.
- Centralizing common agent behavior within AgentsMdAgent prevents divergence across agent integrations.
- Adopting a shared internal module provides a clear extension point for adding new agent types.

## Consequences

Positive:
- Ensures consistent agent lifecycle execution and interface adherence across all specialized agents.
- Eliminates duplicate boilerplate and agent orchestration logic across disparate agent implementations.
- Simplifies the introduction of new agent adapters by providing an established foundation.

Negative:
- Tight coupling of all agent implementations to the internal AgentsMdAgent base module interface.
- Breaking changes in AgentsMdAgent propagate immediately to all derived agent modules.
- Risk of monolithic accretion in the shared base module as individual agent requirements evolve.

## Alternatives

- Independent standalone agent implementations without AgentsMdAgent base (rejected)
  Rejected because: Produces widespread code duplication across agent integrations and causes divergent execution behavior.
  When valid: Valid only if an agent integration has an incompatible execution protocol that cannot conform to the base module contract.
- Composition-based delegate helpers instead of shared base module inheritance (rejected)
  Rejected because: Deviates from established codebase conventions where agent modules share a foundational class contract.
  When valid: Valid when multiple distinct behavioral capabilities need dynamic runtime assembly across heterogeneous agent types.

## Risks

- Base module modifications in AgentsMdAgent may inadvertently regress multiple dependent agent implementations.
  Mitigation: Maintain thorough automated regression test suites covering each derived agent module to detect contract regressions.
  Owner: engineering team
- Agent-specific requirements could leak into AgentsMdAgent, bloating the shared base module.
  Mitigation: Enforce strict code review standards keeping AgentsMdAgent focused strictly on generic agent lifecycle concerns.
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
- When authoring a new agent module, extend AgentsMdAgent and implement specialized hooks rather than reimplementing execution workflows.
- Verify that internal module imports maintain subsystem boundaries and adhere to shared agent interface expectations.

## Continuation Context


Verify commands:
- Discover and run the project static analysis suite to verify that agent modules import AgentsMdAgent.
- Discover and run the project test suite to validate that all agent implementations satisfy regression and integration tests.

Accept when:
- All agent integration modules resolve and import AgentsMdAgent without static analysis errors.
- The project test runner executes and passes all test suites covering the agent subsystem.

## Enforcement

- Verified by: Automated static analysis checks in continuous integration verifying import patterns.
- Verified by: Architecture and code reviews for pull requests modifying or adding agent modules.
- Violation handling: Pull requests lacking base module inheritance in agent components are blocked.
- Violation handling: Violations require refactoring the agent implementation to extend AgentsMdAgent.
- Exception process: Submit an architecture review request detailing why an agent cannot derive from AgentsMdAgent.
- Exception process: Obtain sign-off from module maintainers before introducing an independent agent implementation.