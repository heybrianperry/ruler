# Adopt AgentsMdAgent Base Class for Agent Implementations: Agent Implementation Classes Agents Subsystem Extend

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent implementations in the agents subsystem.

## Context

- The agents subsystem contains multiple agent implementation classes (ZedAgent, CodexCliAgent, GeminiCliAgent, FactoryDroidAgent, CopilotAgent, WindsurfAgent) that require consistent interface and behavior.
- A shared base class pattern was established to enforce common agent capabilities, lifecycle management, and integration points across all agent types.
- Agent implementations need access to shared utilities for filesystem operations, configuration parsing, and metadata management provided by the base class.
- The architecture follows a plugin/strategy pattern where each agent type provides specialized behavior while conforming to a common abstraction.

## Problem Statement

Without a standardized base class, agent implementations would duplicate common functionality, diverge in interface contracts, and create maintenance burden when evolving shared agent capabilities. The system needs a consistent abstraction layer that all agent types must conform to while allowing specialized behavior.

## Decision

1. MUST: All agent implementation classes in the agents subsystem MUST extend or compose the AgentsMdAgent base class to ensure interface consistency and shared capability access.

## Policy Block

- MUST All agent implementation classes in the agents subsystem MUST extend or compose the AgentsMdAgent base class to ensure interface consistency and shared capability access.

In scope:
- All TypeScript classes in the src/agents/ directory that implement agent behavior
- New agent types being added to the agents subsystem
- Refactoring of existing agent implementations

Out of scope:
- Utility classes and helper functions that do not represent agent implementations
- Test fixtures and mock agents used exclusively in test suites
- Agent-related types, interfaces, or configuration objects that do not implement agent behavior

## Rationale

- Static analysis detected AgentsMdAgent imports across 6 agent implementation files with significance scores ranging from 0.90 to 0.93, indicating an established architectural pattern.
- The consistent import pattern across all agent types demonstrates intentional standardization on a shared base class abstraction.
- Centralizing common agent functionality in a base class reduces code duplication and ensures consistent behavior across the agent subsystem.
- The pattern enables polymorphic agent handling and simplifies addition of new agent types through inheritance or composition.

## Consequences

Positive:
- Consistent interface contract across all agent implementations enables polymorphic usage and simplified agent registry management
- Shared functionality centralized in base class reduces code duplication and maintenance burden
- New agent types can be added quickly by extending the base class and overriding specialized methods
- Changes to common agent behavior can be implemented once in the base class rather than across multiple implementations

Negative:
- Tight coupling to AgentsMdAgent base class creates dependency that affects all agent implementations when base class changes
- Base class modifications require careful consideration of impact across all derived agent types
- Inheritance hierarchy may become complex if multiple levels of abstraction are needed
- Agent implementations with significantly different requirements may be forced into an ill-fitting abstraction

## Alternatives

- Use interface-only contract (IAgent) without shared base class implementation (rejected)
  Rejected because: Would require duplicating common functionality across all agent implementations, increasing maintenance burden and risk of behavioral inconsistency
  When valid: When agent implementations have no shared behavior and only need to conform to a type contract
- Use composition with utility modules instead of inheritance-based base class (rejected)
  Rejected because: Evidence shows consistent base class import pattern rather than utility composition; changing would require refactoring all 6 existing implementations
  When valid: For new subsystems where inheritance hierarchy is not yet established and composition provides more flexibility
- Allow each agent to implement its own abstraction without shared base (rejected)
  Rejected because: Would eliminate interface consistency and prevent polymorphic agent handling, fragmenting the agent subsystem architecture
  When valid: Never valid within the agents subsystem; only for completely independent components outside the agent architecture

## Risks

- Base class becomes bloated with specialized logic that only applies to subset of agent types
  Mitigation: Regularly review base class methods for generality; extract specialized behavior into mixins or separate utility modules when only subset of agents need it
  Owner: engineering team
- Breaking changes to AgentsMdAgent base class require coordinated updates across all 6+ agent implementations
  Mitigation: Use semantic versioning for base class changes; provide deprecation warnings and migration paths; maintain backward compatibility where possible
  Owner: engineering team
- Deep inheritance hierarchy may emerge if agent subtypes need further specialization
  Mitigation: Limit inheritance depth to two levels (AgentsMdAgent → concrete agent); use composition for additional specialization beyond base class
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
- Examine the AgentsMdAgent base class implementation to understand the interface contract, lifecycle methods, and shared utilities available to derived classes.
- When creating a new agent type, start by extending AgentsMdAgent and overriding only the methods that require specialized behavior for that agent type.
- Review existing agent implementations (ZedAgent, CodexCliAgent, GeminiCliAgent, etc.) as reference examples for common patterns in extending the base class.

## Continuation Context


Verify commands:
- Discover the project's static analysis or linting configuration and execute the verification script to check that all agent implementation files import AgentsMdAgent
- Discover the project's type checking tooling and verify that all classes in the agents directory conform to the base class interface contract
- Discover the project's test suite and run agent subsystem tests to verify that all agent implementations satisfy the shared interface requirements

Accept when:
- All agent implementation files in the agents directory successfully import and extend or compose AgentsMdAgent base class
- Type checking passes with no interface contract violations in agent implementations
- Agent subsystem tests pass, confirming all implementations satisfy shared interface requirements

## Enforcement

- Verified by: Static analysis tools verify AgentsMdAgent import presence in all agent implementation files
- Verified by: Type checking enforces interface contract compliance at build time
- Verified by: Code review process checks that new agent types extend the base class
- Verified by: Automated tests validate that agent implementations conform to expected interface
- Violation handling: Build fails if type checking detects interface contract violations
- Violation handling: Code review blocks merge of agent implementations that do not extend AgentsMdAgent
- Violation handling: Static analysis warnings flag missing base class imports in agent files
- Exception process: Exceptions require architecture review and documentation of why the agent cannot conform to the base class pattern
- Exception process: Exception approval must come from engineering lead with rationale recorded in code comments
- Exception process: Approved exceptions must document alternative interface contract and integration approach