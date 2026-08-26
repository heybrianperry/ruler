# Centralized IAgent Interface Module for Agent Contracts: Iagent Interface Module Located Shared Location

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The system architecture requires coordination between multiple core infrastructure modules (configuration utilities, MCP capabilities, revert engine, agent selection) and various agent implementations.
- A shared contract is needed to enable polymorphic handling of agents across different subsystems without coupling infrastructure code to specific agent implementations.
- The IAgent interface module serves as the central type definition that establishes behavioral contracts for all agent implementations in the system.
- Core infrastructure modules import this interface to type-check agent interactions, enabling compile-time verification of contract compliance.

## Problem Statement

Core infrastructure modules need to coordinate agent behavior across configuration loading, capability detection, revert operations, and selection logic without creating tight coupling to specific agent implementations. Without a shared contract, each infrastructure module would either duplicate type definitions or depend directly on concrete agent classes, leading to fragile dependencies and reduced extensibility.

## Decision

1. MUST: The IAgent interface module MUST be located in a shared location accessible via relative import from core infrastructure modules.

## Policy Block

- MUST The IAgent interface module MUST be located in a shared location accessible via relative import from core infrastructure modules.

In scope:
- Core infrastructure modules that coordinate agent behavior (configuration, selection, capability detection, lifecycle management)
- Modules in the core and MCP subsystems that interact with agents polymorphically
- Any module that needs to type-check agent contracts without depending on concrete implementations

Out of scope:
- Agent implementation modules themselves (these implement the interface rather than import it as a dependency)
- Modules that interact with a single known agent type and do not require polymorphic handling
- Utility modules that operate on agent data structures without invoking agent behavior

## Rationale

- The evidence shows consistent import of the IAgent interface across 4 distinct core infrastructure modules (config-utils, mcp/capabilities, revert-engine, agent-selection), demonstrating a deliberate architectural pattern rather than isolated usage.
- Co-location of IAgent imports with configuration loading (ConfigLoader), utility functions (agent-utils), and file system operations indicates the interface serves as a contract for infrastructure that coordinates agent behavior without implementing agents.
- The relative import pattern '../agents/IAgent' establishes a clear dependency direction: core infrastructure depends on the interface definition, while agent implementations (in the agents directory) implement the contract, preventing circular dependencies.
- This pattern enables extensibility by allowing new agent implementations to be added without modifying core infrastructure code, as long as they satisfy the IAgent contract.

## Consequences

Positive:
- Core infrastructure modules can coordinate agent behavior polymorphically without coupling to specific agent implementations, improving extensibility.
- Compile-time type checking ensures that infrastructure code correctly interacts with agent contracts, catching integration errors early.
- Clear separation of concerns: infrastructure modules handle coordination logic while agent modules handle implementation logic, with the interface as the boundary.
- New agent types can be added by implementing the IAgent interface without modifying existing infrastructure code.

Negative:
- Changes to the IAgent interface contract require coordinated updates across all infrastructure modules that depend on it and all agent implementations.
- The centralized interface creates a single point of coupling for agent-related contracts, potentially limiting flexibility if different subsystems need divergent agent capabilities.
- Infrastructure modules must handle the full generality of the IAgent interface even if they only use a subset of its capabilities.
- Debugging agent coordination issues may require tracing through the interface abstraction layer to understand concrete behavior.

## Alternatives

- Duplicate agent type definitions locally in each infrastructure module (rejected)
  Rejected because: Creates maintenance burden and type inconsistency across modules, violating DRY principle and making contract evolution difficult.
  When valid: Only valid for prototyping or when modules have genuinely different agent contract requirements that cannot be unified.
- Have infrastructure modules depend directly on concrete agent implementation classes (rejected)
  Rejected because: Creates tight coupling between infrastructure and implementations, preventing polymorphic handling and making it impossible to add new agent types without modifying infrastructure code.
  When valid: Only valid when the system will only ever have a single agent implementation and extensibility is not a requirement.
- Use structural typing without an explicit shared interface module (rejected)
  Rejected because: Loses explicit contract documentation and makes it harder to discover what capabilities agents must provide; structural typing alone does not communicate architectural intent.
  When valid: Valid in dynamically-typed languages or when agent contracts are extremely simple and unlikely to evolve.

## Risks

- Interface bloat: the IAgent interface may accumulate methods over time as different subsystems add requirements, creating a large contract that individual agents struggle to implement fully.
  Mitigation: Periodically review the interface for cohesion; consider splitting into multiple focused interfaces (e.g., IConfigurableAgent, IRevertableAgent) using interface composition if the contract grows too large.
  Owner: engineering team
- Breaking changes to the IAgent interface require coordinated updates across multiple modules and all agent implementations, creating deployment coordination challenges.
  Mitigation: Use versioning or deprecation strategies for interface evolution; consider additive changes (new optional methods) over breaking changes; maintain comprehensive test coverage for interface contracts.
  Owner: engineering team
- The relative import path '../agents/IAgent' creates implicit assumptions about directory structure that may break during refactoring or monorepo reorganization.
  Mitigation: Consider using path aliases or module resolution configuration to decouple import paths from physical file structure; document the expected module layout in architecture documentation.
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
- When creating new core infrastructure modules that coordinate agents, import the IAgent interface at the top of the module alongside other internal dependencies, using the established relative import pattern from the module's location to the interface definition.
- When the IAgent interface needs to evolve, assess the impact on all importing modules by searching the codebase for interface imports before making changes; prefer additive changes (new optional methods or properties) over breaking changes to existing contracts.
- Infrastructure modules should type function parameters and return values using the IAgent interface type to ensure compile-time verification of contract compliance; avoid using 'any' or overly generic types that bypass the contract.

## Continuation Context


Verify commands:
- Discover the project's module resolution configuration and verify that the IAgent interface module is accessible from core infrastructure locations using the documented import pattern.
- Search the codebase for all imports of the IAgent interface and verify that importing modules are in the core infrastructure scope (configuration, selection, capability detection, lifecycle management).
- Run the project's type checking tooling to verify that all IAgent interface usage in infrastructure modules satisfies the contract without type errors.

Accept when:
- All core infrastructure modules that coordinate agent behavior successfully import and use the IAgent interface type without compilation or type-checking errors.
- No duplicate agent interface definitions exist in infrastructure modules; the centralized IAgent module is the single source of truth for agent contracts.
- Agent implementation modules implement the IAgent interface while infrastructure modules depend on it, establishing the correct dependency direction.

## Enforcement

- Verified by: Static type checking during continuous integration verifies that all IAgent interface usage satisfies the contract.
- Verified by: Code review checks that new infrastructure modules use the centralized interface rather than duplicating definitions or depending on concrete implementations.
- Verified by: Automated dependency analysis tools can verify the import graph maintains the correct dependency direction (infrastructure → interface ← implementations).
- Violation handling: Type checking failures that indicate incorrect IAgent interface usage block the build and must be resolved before merge.
- Violation handling: Code review identifies and rejects changes that duplicate the interface or create direct dependencies on concrete agent classes.
- Violation handling: Violations discovered in existing code should be refactored to use the centralized interface pattern during the next maintenance cycle for the affected module.
- Exception process: Exceptions may be granted for agent implementation modules that legitimately need to implement rather than import the interface.
- Exception process: Temporary exceptions may be granted during major refactoring efforts where the interface is being restructured, with a documented plan to restore compliance.
- Exception process: Exception requests must document why the centralized interface pattern cannot be used and what alternative approach maintains contract-based coordination.