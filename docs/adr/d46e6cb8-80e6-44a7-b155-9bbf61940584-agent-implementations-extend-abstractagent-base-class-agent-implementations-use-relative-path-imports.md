# Agent Implementations Extend AbstractAgent Base Class: Agent Implementations Use Relative Path Imports

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent implementations in the agents module.

## Context

- The project implements multiple agent types (FirebaseAgent, ClaudeAgent, WarpAgent, ClineAgent, TraeAgent) that require consistent behavior and shared functionality across different AI service backends.
- An AbstractAgent base class was introduced to centralize common agent logic and establish a uniform interface for polymorphic agent handling.
- The agents module serves as a plugin-like architecture where different service integrations can be added without modifying core orchestration logic.
- Six files in src/agents/ demonstrate consistent adoption of the AbstractAgent inheritance pattern with high significance (0.90), indicating this is an established architectural convention rather than experimental code.

## Problem Statement

Multiple agent implementations require shared behavior, lifecycle management, and polymorphic treatment. Without a common base class, each agent would duplicate core logic, making maintenance difficult and preventing uniform handling of different agent types in orchestration code. The system needs a way to enforce consistent agent contracts while allowing specialization for different AI service backends.

## Decision

1. MUST: Agent implementations MUST use relative path imports to reference AbstractAgent from within the agents module.

## Policy Block

- MUST Agent implementations MUST use relative path imports to reference AbstractAgent from within the agents module.

In scope:
- All TypeScript classes in the agents module that implement agent behavior for AI service backends
- New agent implementations added to support additional AI services or orchestration patterns
- Refactoring of existing agent code within the agents module

Out of scope:
- Utility classes or helper modules in the agents directory that do not represent agent implementations
- Agent client code that consumes agents through the IAgent interface
- Test doubles or mocks that may use alternative inheritance structures for testing purposes

Exceptions:
- EX-001: A specialized agent requires fundamentally different lifecycle or contract that cannot be accommodated by AbstractAgent

## Rationale

- The evidence shows six files consistently importing and extending AbstractAgent with high pattern significance (0.90), demonstrating this is an established architectural convention.
- Inheritance-based polymorphism enables uniform handling of different agent types in orchestration code while allowing specialization for specific AI service backends (Firebase, Claude, Warp, Cline, Trae).
- Centralizing shared agent logic in AbstractAgent reduces code duplication and ensures consistent behavior across all agent implementations.
- The presence of both AbstractAgent and IAgent suggests a layered contract approach: AbstractAgent provides implementation inheritance while IAgent defines the public interface contract.

## Consequences

Positive:
- New agent types can be added quickly by extending AbstractAgent and implementing only backend-specific logic.
- Orchestration code can treat all agents polymorphically through the common base class or IAgent interface.
- Shared agent behavior (lifecycle, error handling, state management) is centralized and maintained in one location.
- The pattern enforces consistent agent contracts, reducing integration bugs and improving maintainability.

Negative:
- Inheritance coupling means changes to AbstractAgent may impact all agent implementations, requiring coordinated updates.
- Deep inheritance hierarchies can make behavior harder to trace and debug compared to composition-based approaches.
- Agents with fundamentally different contracts may be forced into an inappropriate abstraction or require exception handling.
- Testing may require more complex setup to properly mock or stub AbstractAgent behavior.

## Alternatives

- Composition-based approach using agent capabilities as injectable dependencies rather than inheritance (rejected)
  Rejected because: The evidence shows established inheritance pattern across six files; composition would require significant refactoring and may not provide sufficient benefit given the current architecture's stability.
  When valid: For new modules or when agent contracts become too diverse to fit a single base class hierarchy
- Interface-only contract (IAgent) without AbstractAgent base class, allowing each agent full implementation freedom (rejected)
  Rejected because: Would result in duplicated common logic across all agent implementations and lose the benefit of centralized behavior management that AbstractAgent provides.
  When valid: When agents have fundamentally different lifecycles or when shared logic is minimal
- Factory pattern with strategy objects for backend-specific behavior instead of inheritance (rejected)
  Rejected because: The current inheritance pattern is well-established and provides clear specialization points; factory pattern would add indirection without clear architectural benefit.
  When valid: If runtime agent type selection or dynamic backend switching becomes a requirement

## Risks

- AbstractAgent becomes a god class accumulating too much responsibility as new agent types are added
  Mitigation: Regularly review AbstractAgent for single responsibility violations; extract shared concerns into separate mixins or utility modules when appropriate
  Owner: engineering team
- Breaking changes to AbstractAgent API require coordinated updates across all agent implementations
  Mitigation: Use semantic versioning for AbstractAgent changes; provide deprecation warnings and migration paths for breaking changes; maintain comprehensive test coverage
  Owner: engineering team
- New agent types with incompatible contracts may force awkward workarounds or violate Liskov Substitution Principle
  Mitigation: Document clear criteria for when an agent should extend AbstractAgent versus implementing IAgent directly; establish exception process for divergent cases
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
- When creating a new agent implementation, examine AbstractAgent to understand which methods are abstract (must be implemented) versus concrete (can be inherited). Override only what is necessary for your specific backend.
- The agents module index serves as the single export point for all agent types. Add new agents to this index to ensure they are discoverable by orchestration code.
- Consider whether your agent implementation requires state management, lifecycle hooks, or error handling patterns that AbstractAgent may already provide before implementing custom solutions.

## Continuation Context


Verify commands:
- Discover the agents module location in the repository and inspect all class files to confirm they import and extend the base class
- Locate the project's static analysis or linting configuration and execute the type checker to verify inheritance contracts are satisfied
- Identify the test suite for the agents module and run tests to confirm polymorphic behavior works correctly across all agent implementations

Accept when:
- All agent implementation files in the agents module contain import statements referencing the base class and use extends syntax
- Type checking passes without errors related to missing abstract method implementations or contract violations
- Agent module tests demonstrate that all concrete agent types can be instantiated and used polymorphically through the common interface

## Enforcement

- Verified by: TypeScript compiler type checking enforces that agent classes properly extend AbstractAgent and implement required abstract methods
- Verified by: Code review process verifies new agent implementations follow the inheritance pattern
- Verified by: Static analysis tools can detect direct IAgent implementations that bypass AbstractAgent
- Violation handling: TypeScript compilation will fail if an agent class does not properly extend AbstractAgent or implement required methods
- Violation handling: Code review should reject pull requests that add agent implementations without following the established pattern
- Violation handling: Document any approved exceptions in the agent implementation file with clear rationale
- Exception process: Developer identifies a legitimate case where AbstractAgent inheritance is inappropriate for a new agent type
- Exception process: Submit architecture review request documenting why the standard pattern cannot accommodate the new agent's requirements
- Exception process: If approved, document the exception in the agent implementation file and update this ADR's exception list