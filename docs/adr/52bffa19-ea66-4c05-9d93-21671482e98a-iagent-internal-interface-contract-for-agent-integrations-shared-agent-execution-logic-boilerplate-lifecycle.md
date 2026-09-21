# IAgent Internal Interface Contract for Agent Integrations: Shared Agent Execution Logic Boilerplate Lifecycle

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The system supports multiple distinct agent integrations targeting diverse developer environments, command-line interfaces, and editor workflows.
- Static analysis of the agent subsystem confirms universal adoption of the internal IAgent contract across concrete agent classes and helper utilities.
- A standardized interface boundary is required to decouple consumer orchestration and registry indexing from concrete agent integration mechanics.

## Problem Statement

Without a standardized contract across agent implementations, divergent lifecycle methods and ad-hoc public interfaces prevent polymorphic dispatch and create tight coupling between callers and specific agent environments.

## Decision

1. SHOULD: Shared agent execution logic and boilerplate lifecycle handling SHOULD be factored into abstract base modules conforming to IAgent to prevent redundant code across concrete implementations.

## Policy Block

- SHOULD Shared agent execution logic and boilerplate lifecycle handling SHOULD be factored into abstract base modules conforming to IAgent to prevent redundant code across concrete implementations.

In scope:
- All agent integration modules, agent adapters, and agent utility implementations within the agent subsystem.

Out of scope:
- Generic core utilities, standalone file system helpers, and non-agent domain components that do not manage agent lifecycles.

## Rationale

- Static analysis confirms that all nine analyzed agent implementations and utility modules import the IAgent interface, establishing it as the authoritative contract for agent behaviors.
- Adhering to the IAgent contract decouples orchestrators and registry modules from the concrete execution and configuration parsing details of individual agent targets.
- A unified contract simplifies testing and mock substitution while providing a clear integration template for future agent implementations.

## Consequences

Positive:
- Enables polymorphic invocation and unified lifecycle orchestration across heterogeneous agent implementations.
- Simplifies agent discovery, indexing, and registration within central registry modules.
- Encapsulates environment-specific parsing and configuration routines within concrete agent modules behind a stable boundary.

Negative:
- Constrains individual agent implementations to the lifecycle and method boundaries dictated by the IAgent contract.
- Requires coordinated updates across all implementing agent classes whenever the shared contract evolves.

## Alternatives

- Ad-hoc standalone agent classes without a shared interface contract (rejected)
  Rejected because: Leads to divergent public APIs, increases cognitive load for consumers, and prevents managing agents polymorphically via a unified registry.
  When valid: Only viable in minimal prototypes containing a single agent implementation where polymorphic invocation is unnecessary.
- Rigid single-class inheritance hierarchy without interface abstraction (rejected)
  Rejected because: Couples all agent implementations to rigid inheritance chains and prevents flexible composition for diverse external agent protocols.
  When valid: Valid when all agent integrations share identical execution mechanics and require no specialized behavior or composition.

## Risks

- Interface bloat if provider-specific requirements are continually pushed into the shared IAgent contract.
  Mitigation: Keep the IAgent interface focused strictly on common lifecycle hooks, using composition or optional capability interfaces for specialized provider behaviors.
  Owner: Engineering Team
- Breaking changes to the IAgent contract necessitating simultaneous refactoring across all concrete agent implementations.
  Mitigation: Provide backward-compatible abstract base implementations with default handling whenever expanding the contract.
  Owner: Engineering Team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When creating a new agent integration, implement the IAgent contract and register the implementation within the primary agent registry index module.
- Agent integrations requiring specialized configuration parsing must encapsulate parsing logic within private implementation methods while exposing standard IAgent lifecycle behaviors.

## Continuation Context


Verify commands:
- Discover and execute the repository static type checker and linter across the agent subsystem to verify interface conformance with the IAgent contract.
- Discover and run the project test suite targeting agent integration test suites to validate contract adherence.

Accept when:
- Static type analysis confirms that all concrete agent integration modules implement the IAgent interface contract without type errors.
- The project test suite passes with zero failures across all agent integration and utility test suites.

## Enforcement

- Verified by: Automated static type analysis and linting within continuous integration pipelines verifying interface conformance.
- Verified by: Peer code review for all new or modified agent integration modules.
- Violation handling: Pull requests introducing agent implementations that do not conform to the IAgent contract are blocked from merging.
- Violation handling: Static analysis failures during continuous integration trigger immediate build termination.
- Exception process: Architectural exceptions require formal review and documented approval by the agent subsystem maintainers.
- Exception process: Any temporary deviation must be tracked with an issue defining the remediation timeline for interface compliance.