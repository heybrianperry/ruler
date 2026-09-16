# Adoption of Core IAgent Interface for Agent Management: Leverage Agent Utils Common Related Utility

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- Consistent use of an IAgent module across core functionalities.
- Integration with configuration loading and agent utility functions.
- The need for a standardized approach to agent interactions within the system.
- Centralization of agent-related logic and contracts.

## Problem Statement

The codebase requires a consistent and well-defined interface for managing and interacting with various agent implementations and their configurations across different core modules.

## Decision

1. SHOULD: SHOULD leverage agent-utils for common agent-related utility functions.

## Policy Block

- SHOULD SHOULD leverage agent-utils for common agent-related utility functions.

In scope:
- Modules within src/core and src/mcp that deal with agent definition, selection, configuration, or capabilities.

Out of scope:
- Modules outside src/core and src/mcp that do not directly interact with agent interfaces.

## Rationale

- Standardizes agent interactions, promoting consistency and reducing integration complexity.
- Centralizes agent-related concerns, improving maintainability and testability.
- Provides a clear contract for future agent implementations and extensions.
- Leverages existing internal modules for configuration and utilities, avoiding re-invention.

## Consequences

Positive:
- Improved code clarity and maintainability for agent-related modules.
- Easier onboarding for new developers working on agent functionalities.
- Consistent behavior and interaction patterns across different agents.
- Facilitates unit testing and mocking of agent dependencies.

Negative:
- Strict adherence to the IAgent interface might introduce overhead for simple agent implementations.
- Changes to the IAgent interface could impact multiple dependent modules.

## Alternatives

- Define agent contracts ad-hoc using individual classes/functions. (rejected)
  Rejected because: Leads to inconsistent patterns, increased boilerplate, and reduced maintainability.
  When valid: For very small, isolated projects with no need for standardized agent interactions.
- Use a third-party dependency injection framework for agent management. (rejected)
  Rejected because: Introduces external dependency overhead and complexity for a problem already addressed by internal modules.
  When valid: If the internal IAgent module proves insufficient for highly complex dependency graphs or advanced DI features.

## Risks

- Over-engineering for simple agent tasks.
  Mitigation: Ensure the IAgent interface remains lean and focused on core agent responsibilities.
  Owner: Engineering team.
- Breaking changes to IAgent impacting many modules.
  Mitigation: Implement strict versioning and deprecation policies for IAgent and related core modules.
  Owner: Engineering team.

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- New agent implementations should start by defining a class that implements the IAgent interface.
- Consider creating a dedicated agents directory within src/core for all agent-related implementations.

## Continuation Context


Verify commands:
- Discover and run the project's static analysis tools.
- Discover and run the project's unit tests for agent-related modules.
- Discover and run the project's integration tests involving agent interactions.

Accept when:
- Static analysis reports no violations of IAgent interface usage.
- All agent-related unit and integration tests pass.
- New agent implementations correctly adhere to the IAgent contract.

## Enforcement

- Verified by: Automated CI checks and code reviews.
- Violation handling: CI pipeline failure, mandatory code review comments requiring adherence.
- Exception process: Formal review by architectural governance board with documented rationale.