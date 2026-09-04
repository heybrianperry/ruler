# Adopt IAgent Interface as Canonical Type Contract for Agent Coordination: Infrastructure Components That Coordinate Select Configure

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all infrastructure components that coordinate, select, configure, or manage agent lifecycle.

## Context

- The project implements multiple agent types with heterogeneous capabilities and behaviors that require coordination by infrastructure components.
- Infrastructure components including MCP capabilities, agent selection logic, configuration utilities, and revert engine need to interact with agents polymorphically without coupling to specific implementations.
- A shared type contract enables type-safe agent coordination while maintaining decoupling between infrastructure and agent implementation details.
- The IAgent interface module is located in the agents package but consumed by core infrastructure, establishing a clear dependency direction from infrastructure to agent contracts.

## Problem Statement

Infrastructure components that coordinate multiple agent types need a stable, type-safe contract for agent interactions without coupling to specific agent implementations. Without a canonical interface, infrastructure code would either depend on concrete agent classes (creating tight coupling) or use untyped references (losing type safety and IDE support).

## Decision

1. MUST: Infrastructure components that coordinate, select, configure, or manage agent lifecycle MUST import and type agent references against the IAgent interface from the agents module.

## Policy Block

- MUST Infrastructure components that coordinate, select, configure, or manage agent lifecycle MUST import and type agent references against the IAgent interface from the agents module.

In scope:
- MCP capability implementations that expose agent functionality
- Agent selection and routing logic
- Configuration utilities that load or validate agent configurations
- Lifecycle management components including revert engines and apply engines
- Any infrastructure component that coordinates multiple agent types polymorphically

Out of scope:
- Agent implementation classes themselves (these implement the interface rather than import it for coordination)
- Agent-specific utility functions that operate on a single known agent type
- Test fixtures that instantiate concrete agent types for testing purposes
- Components that interact with agents through higher-level abstractions that already encapsulate the IAgent dependency

## Rationale

- The pattern is observed across 4 infrastructure files spanning MCP integration, agent selection, configuration, and revert engine subsystems, indicating an established architectural convention with significance 0.90.
- Using a shared interface contract enables polymorphic agent coordination while maintaining decoupling between infrastructure and agent implementations, supporting extensibility as new agent types are added.
- The consistent relative import pattern from infrastructure to the agents module establishes a clear dependency direction that prevents circular dependencies and maintains module boundaries.
- Type safety at the coordination layer reduces runtime errors and improves developer experience through IDE autocomplete and compile-time validation of agent interactions.

## Consequences

Positive:
- Infrastructure components can coordinate heterogeneous agent types through a stable, type-safe contract without coupling to implementation details.
- New agent types can be added by implementing the IAgent interface without modifying infrastructure code, supporting the Open-Closed Principle.
- Type safety enables compile-time detection of contract violations and provides IDE support for agent coordination code.
- Clear dependency direction from infrastructure to agent contracts prevents circular dependencies and maintains clean module boundaries.

Negative:
- Changes to the IAgent interface contract require coordinated updates across all infrastructure consumers and agent implementations.
- The interface may become a bottleneck if it needs to accommodate divergent agent capabilities, potentially leading to interface bloat or optional methods.
- Relative path imports create coupling to the physical file structure, requiring updates if the agents module is relocated.
- Infrastructure components cannot leverage agent-specific capabilities beyond the IAgent contract without type assertions or additional patterns.

## Alternatives

- Use concrete agent classes directly in infrastructure components (rejected)
  Rejected because: Creates tight coupling between infrastructure and specific agent implementations, violating the Dependency Inversion Principle and making it difficult to add new agent types without modifying infrastructure code.
  When valid: In single-agent systems where polymorphism is not required and the infrastructure is tightly coupled to one agent implementation by design.
- Use untyped references or generic object types for agent coordination (rejected)
  Rejected because: Loses type safety and IDE support, increasing the risk of runtime errors and reducing developer productivity when working with agent coordination code.
  When valid: In dynamically-typed contexts or when agent capabilities are so heterogeneous that a shared interface cannot capture meaningful commonality.
- Define the IAgent interface in a separate shared types package rather than in the agents module (deferred)
  Rejected because: Not rejected; this is a valid alternative that would further decouple the interface from agent implementations. The current pattern co-locates the interface with agents, which is simpler but creates a dependency on the agents module.
  When valid: When the project grows to include multiple packages that need to reference agent contracts without depending on the agents module, or when the interface needs to be versioned independently.

## Risks

- Interface bloat as new agent types require additional methods or properties in the IAgent contract, leading to a complex interface that not all agents can meaningfully implement.
  Mitigation: Use interface segregation to split IAgent into focused sub-interfaces if capabilities diverge significantly. Consider optional methods or capability detection patterns for agent-specific features.
  Owner: engineering team
- Breaking changes to the IAgent interface require coordinated updates across all infrastructure consumers and agent implementations, creating migration complexity.
  Mitigation: Version the interface or use extension interfaces for new capabilities. Maintain backward compatibility through optional properties or default implementations where possible.
  Owner: engineering team
- Relative path imports couple infrastructure to the physical file structure, requiring updates if the agents module is relocated or the project structure changes.
  Mitigation: Consider migrating to path aliases or module resolution configuration that decouples logical imports from physical structure. Document the import convention clearly.
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
- When creating new infrastructure components that coordinate agents, import the IAgent interface using the relative path convention observed in existing components. Co-locate the import with other infrastructure dependencies for consistency.
- If an infrastructure component needs agent-specific capabilities beyond the IAgent contract, consider using type guards or capability detection patterns rather than importing concrete agent classes.
- When modifying the IAgent interface, audit all infrastructure consumers to ensure compatibility. Use the project's type checking tooling to identify breaking changes before committing.

## Continuation Context


Verify commands:
- Discover the project's static analysis configuration and execute the type checking tool to verify that all infrastructure components importing IAgent have valid type references.
- Locate the project's module dependency analysis tooling and verify that infrastructure components do not import concrete agent implementation classes for type declarations.
- Identify the project's linting configuration and run the linter to verify that relative import paths to IAgent follow the established convention.

Accept when:
- Type checking passes with no errors related to IAgent interface usage in infrastructure components.
- Dependency analysis confirms that infrastructure components depend only on the IAgent interface, not on concrete agent implementation classes.
- All relative imports to IAgent follow the established path convention and resolve correctly.

## Enforcement

- Verified by: Automated type checking in continuous integration pipeline
- Verified by: Code review verification that new infrastructure components follow the IAgent import pattern
- Verified by: Static analysis tooling that detects imports of concrete agent classes in infrastructure components
- Violation handling: Type checking failures block merge until infrastructure components correctly type against IAgent
- Violation handling: Code review feedback requests refactoring of infrastructure components that import concrete agent classes
- Violation handling: Static analysis warnings are escalated to errors for infrastructure components that violate the interface dependency rule
- Exception process: Exceptions require architectural review and documentation of why the IAgent contract is insufficient for the use case
- Exception process: Approved exceptions must document the specific agent capability required and why it cannot be added to the IAgent interface
- Exception process: Exception approval includes a plan for either extending the IAgent interface or refactoring the component to work within the contract