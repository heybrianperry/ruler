# AgentsMdAgent Internal Module Adoption for Agent Adapters: Prior Introducing Utilizing Any External Parsing

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The codebase implements multiple distinct agent adapters to interface with external AI coding assistants and CLI tools.
- Diverse agent implementations share common responsibilities including configuration handling, markdown agent representation, and standardized lifecycle management.
- Static analysis identifies consistent imports of the internal AgentsMdAgent base module and the IAgent interface across all agent adapter files.

## Problem Statement

Integrating diverse external AI agent tools without a unified core abstraction leads to divergent configuration handling, inconsistent agent lifecycle state, and fragmented filesystem operations across adapters. An architectural standard is required to enforce interface consistency and shared configuration capabilities across all agent adapters.

## Decision

1. MUST: Prior to introducing or utilizing any external parsing or serialization dependencies for agent configuration, the consumer MUST inspect the project lock artifact to resolve and verify the exact dependency version against documented interface contracts.

## Policy Block

- MUST Prior to introducing or utilizing any external parsing or serialization dependencies for agent configuration, the consumer MUST inspect the project lock artifact to resolve and verify the exact dependency version against documented interface contracts.

In scope:
- All current and future agent adapter implementations in the agent integration subsystem.
- Modules providing interfaces or base classes for AI agent interaction and markdown configuration.

Out of scope:
- General application utilities and core filesystem helpers that do not define or wrap agent adapters.
- External orchestration scripts that consume the unified agent interface without defining agent adapters.

## Rationale

- Eight distinct agent adapter files uniformly import the internal AgentsMdAgent module and implement the IAgent contract, proving an established cross-cutting structural convention.
- Centralizing common agent markdown operations and configuration parsing into a shared internal module minimizes boilerplate across third-party tool adapters.
- Standardizing on the IAgent contract ensures that higher-level orchestration layers can interact with any agent adapter polymorphically.

## Consequences

Positive:
- Standardizes agent adapter lifecycles and configuration management through a single reusable base module.
- Enforces polymorphic interoperability across all agent tools via the shared IAgent interface.
- Decreases boilerplate code in individual agent adapters by encapsulating common markdown parsing and filesystem interactions.

Negative:
- Couples all agent adapters directly to the internal AgentsMdAgent base class implementation.
- Modifications to the shared base module risk breaking multiple agent adapters simultaneously if contracts change.

## Alternatives

- Direct IAgent interface implementation without a shared AgentsMdAgent base class (rejected)
  Rejected because: Forcing each agent adapter to implement markdown formatting, metadata resolution, and configuration persistence independently creates widespread code duplication and inconsistent agent state representation across implementations.
  When valid: Valid only if agent adapters share zero common markdown parsing, lifecycle methods, or file persistence structures.
- Functional composition of agent utilities without inheritance (deferred)
  Rejected because: The existing codebase standardizes agent behaviors through class-based inheritance and shared interface contracts across all detected agent adapters.
  When valid: Valid if the architectural design migrates from object-oriented agent classes to functional composition.

## Risks

- Changes to the internal AgentsMdAgent base module signature or lifecycle hooks could cause regressions across all concrete agent adapters.
  Mitigation: Maintain comprehensive test coverage for base class methods and run continuous integration validation for all derived agent adapters on every modification.
  Owner: Agent platform engineering team
- Over-reliance on base class defaults may obscure tool-specific configuration requirements or security constraints in individual agent adapters.
  Mitigation: Enforce strict schema validation on adapter-specific configuration parsing and require explicit override points for custom agent options.
  Owner: Agent platform engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When creating a new agent adapter, inherit from the internal AgentsMdAgent class and implement all abstract lifecycle and configuration methods mandated by the IAgent contract.
- Delegate generic configuration file parsing and markdown synchronization to base class methods, restricting adapter-specific code to tool-specific options and execution semantics.

## Continuation Context


Verify commands:
- Discover the repository script configuration to identify and execute the static analysis and type verification suite across all agent modules.
- Discover and run the architectural linting and module boundary test suites to confirm that all agent implementations conform to the internal base module and interface contract.

Accept when:
- All agent adapter modules cleanly resolve dependencies and pass type checking while extending the internal AgentsMdAgent module.
- Module boundary checks confirm that no agent adapter module bypasses the IAgent interface or provides duplicate core agent lifecycle implementations.

## Enforcement

- Verified by: Automated continuous integration checks executing static type verification and module boundary lint rules.
- Verified by: Architecture review during pull requests to verify that all new agent adapters derive from the AgentsMdAgent base class.
- Violation handling: Pull requests containing agent adapters that do not inherit from AgentsMdAgent or implement IAgent are blocked from merging.
- Violation handling: Violations identified during code review must be refactored to extend the standard base module before release.
- Exception process: Exceptions require documented architectural approval demonstrating why a standalone agent adapter cannot comply with the standard base contract.