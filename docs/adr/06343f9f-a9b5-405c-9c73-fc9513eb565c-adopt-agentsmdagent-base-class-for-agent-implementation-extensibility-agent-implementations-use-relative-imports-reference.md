# Adopt AgentsMdAgent Base Class for Agent Implementation Extensibility: Agent Implementations Use Relative Imports Reference

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent implementations in the codebase.

## Context

- The codebase implements multiple agent types (CodexCliAgent, ZedAgent, WindsurfAgent, CursorAgent, FactoryDroidAgent, CopilotAgent) that require shared functionality and a common contract.
- An inheritance-based architecture was established using AgentsMdAgent as the base class and IAgent as the interface contract to enable code reuse and enforce consistent agent behavior.
- All six agent implementations in the src/agents directory import and extend AgentsMdAgent, demonstrating a mature and stable architectural pattern.
- The pattern uses relative imports to maintain module cohesion and self-containment within the agents package.
- This architecture supports extensibility by allowing new agent types to be added by extending the base class while inheriting common functionality.

## Problem Statement

The system requires multiple agent implementations with varying capabilities while maintaining consistent behavior, shared functionality, and a uniform interface. Without a common architectural foundation, agent implementations would duplicate code, diverge in behavior, and lack a clear contract for integration with the broader system.

## Decision

1. MUST: Agent implementations MUST use relative imports to reference AgentsMdAgent and IAgent to maintain module cohesion within the agents package.

## Policy Block

- MUST Agent implementations MUST use relative imports to reference AgentsMdAgent and IAgent to maintain module cohesion within the agents package.

In scope:
- All new agent implementations added to the agents package
- Any refactoring or modification of existing agent implementations
- Code that extends or modifies the agent system architecture

Out of scope:
- Utility modules that support agents but are not themselves agent implementations
- Client code that consumes agent functionality through the IAgent interface
- Test fixtures or mock agents used exclusively in test environments

Exceptions:
- EX-001: Creating a fundamentally different agent abstraction that cannot reasonably extend AgentsMdAgent due to incompatible architectural assumptions

## Rationale

- The evidence shows all six agent implementations consistently import and extend AgentsMdAgent, demonstrating this is an established architectural pattern with high adoption (significance 0.90-0.93).
- Inheritance-based extensibility through a shared base class reduces code duplication, ensures consistent behavior across agent types, and provides a single point of maintenance for common functionality.
- The IAgent interface establishes a contract that enables polymorphic usage of agents throughout the system, allowing client code to depend on abstractions rather than concrete implementations.
- Using relative imports within the agents package maintains strong module cohesion and makes the agent system self-contained and easier to reason about.

## Consequences

Positive:
- New agent types can be added quickly by extending AgentsMdAgent and overriding specific methods, reducing implementation time and cognitive load.
- Shared functionality (file system operations, configuration parsing, common workflows) is centralized in the base class, eliminating duplication across agent implementations.
- The IAgent interface enables polymorphic usage, allowing the system to treat all agents uniformly regardless of their concrete type.
- The architectural pattern is self-documenting through the inheritance hierarchy, making it clear to developers how to implement new agents.

Negative:
- Inheritance creates tight coupling between agent implementations and the base class, making it harder to change AgentsMdAgent without affecting all derived classes.
- The base class can become a 'god object' if too much functionality is added, reducing cohesion and making it difficult to understand and maintain.
- Agents with fundamentally different behavior may be forced into an inheritance hierarchy that doesn't fit their needs, leading to awkward overrides or unused inherited functionality.
- Testing becomes more complex as agent implementations inherit behavior that must be understood and potentially mocked during unit testing.

## Alternatives

- Use composition over inheritance by implementing agents as standalone classes that compose shared functionality through injected dependencies or utility modules (rejected)
  Rejected because: The evidence shows all six agents consistently use inheritance, indicating this pattern was evaluated and rejected in favor of the current approach. Composition would require more boilerplate and explicit wiring of shared functionality.
  When valid: When agents have fundamentally different lifecycles or when shared functionality is minimal and better expressed as utility functions
- Implement agents as pure functions or modules without class-based architecture, using functional composition for shared behavior (rejected)
  Rejected because: The evidence shows class-based implementations with inheritance, suggesting the system requires stateful agents with lifecycle management that is better expressed through object-oriented patterns.
  When valid: When agents are stateless transformations or when the system adopts a functional programming paradigm
- Use a plugin architecture where agents register capabilities without inheritance, relying on a registry pattern and capability-based dispatch (rejected)
  Rejected because: The current inheritance pattern provides stronger compile-time guarantees and clearer structure. A plugin architecture would add complexity without clear benefits given the current agent count and stability.
  When valid: When the number of agent types grows significantly (20+) or when third-party developers need to contribute agents without modifying core code

## Risks

- The AgentsMdAgent base class becomes bloated with functionality needed by only some derived classes, reducing cohesion and increasing maintenance burden.
  Mitigation: Regularly review the base class to ensure all methods are used by multiple derived classes. Extract specialized functionality into mixins, utility modules, or composition-based helpers when only one or two agents need it.
  Owner: engineering team
- Changes to AgentsMdAgent or IAgent break multiple agent implementations simultaneously, creating cascading failures and increasing the blast radius of refactoring.
  Mitigation: Maintain comprehensive test coverage for the base class and interface. Use semantic versioning for internal modules if they are shared across packages. Implement integration tests that verify all agent types work correctly after base class changes.
  Owner: engineering team
- New developers may not understand the inheritance hierarchy and either bypass it or use it incorrectly, leading to inconsistent agent implementations.
  Mitigation: Document the agent architecture with clear examples and guidelines. Enforce the pattern through code review and automated linting rules that detect agents not extending AgentsMdAgent. Provide agent implementation templates or generators.
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
- When implementing a new agent, examine the existing agent implementations to understand the methods available in AgentsMdAgent and the contract defined by IAgent. Override only the methods that require agent-specific behavior.
- If an agent requires functionality not present in the base class, first evaluate whether the functionality is general enough to belong in AgentsMdAgent. If it benefits multiple agents, add it to the base class. If it is agent-specific, implement it in the derived class.
- Use TypeScript's type system to ensure agent implementations correctly implement the IAgent interface. Enable strict type checking to catch interface violations at compile time rather than runtime.

## Continuation Context


Verify commands:
- Discover the project's static analysis configuration and execute the type checker to verify all agent implementations correctly extend AgentsMdAgent and implement IAgent
- Discover the project's test runner and execute the test suite for the agents package to verify all agent implementations function correctly with the base class
- Discover the project's linting configuration and execute the linter to verify import patterns and inheritance structure conform to this ADR

Accept when:
- All agent implementations in the agents package extend AgentsMdAgent and the type checker reports no inheritance or interface violations
- All tests for agent implementations pass, demonstrating that the inheritance pattern does not break agent functionality
- Code review confirms that new agent implementations follow the established pattern and do not bypass the base class

## Enforcement

- Verified by: Automated type checking in continuous integration verifies all agents extend AgentsMdAgent and implement IAgent
- Verified by: Code review process checks that new agent implementations follow the inheritance pattern
- Verified by: Automated linting rules detect agents that do not import or extend the base class
- Violation handling: Type checker failures block pull request merges until agents correctly extend the base class
- Violation handling: Code review feedback requires refactoring agents that bypass the inheritance pattern
- Violation handling: Linting violations are treated as build failures and must be resolved before code integration
- Exception process: Developer documents the architectural reason why AgentsMdAgent cannot be extended for a specific agent type
- Exception process: Architecture review evaluates whether the exception is justified or whether the base class should be refactored to accommodate the new agent
- Exception process: If approved, the exception is documented in code comments and in the ADR exception log with the rationale and alternative pattern being used