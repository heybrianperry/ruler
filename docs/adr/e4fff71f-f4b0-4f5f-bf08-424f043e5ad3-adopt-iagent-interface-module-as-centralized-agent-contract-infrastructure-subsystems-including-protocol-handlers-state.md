# Adopt IAgent Interface Module as Centralized Agent Contract: Infrastructure Subsystems Including Protocol Handlers State

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all infrastructure code that interacts with agent abstractions.

## Context

- The project implements a multi-agent architecture where different subsystems (protocol handlers, state management, configuration, selection logic) need to interact with agent abstractions without coupling to concrete implementations.
- A centralized interface module at ../agents/IAgent has been adopted across 4 core infrastructure files with 0.90 significance, indicating an established architectural convention.
- Infrastructure components including MCP capabilities, revert engine, config utilities, and agent selection logic all import this shared interface, creating a consistent contract boundary.
- The pattern enables dependency inversion where core subsystems depend on the IAgent abstraction rather than concrete agent classes, supporting polymorphic agent handling and preventing circular dependencies.

## Problem Statement

Core infrastructure subsystems need to reference agent contracts and interact with agent abstractions without creating tight coupling to concrete agent implementations. Without a centralized interface module, each subsystem might define its own agent contracts, leading to inconsistent abstractions, circular dependencies, and difficulty in maintaining a coherent agent architecture across protocol handlers, state management, configuration, and selection logic.

## Decision

1. MUST: Infrastructure subsystems including protocol handlers, state management, configuration utilities, and agent selection logic MUST reference IAgent when declaring agent-typed parameters, return values, or data structures.

## Policy Block

- MUST Infrastructure subsystems including protocol handlers, state management, configuration utilities, and agent selection logic MUST reference IAgent when declaring agent-typed parameters, return values, or data structures.

In scope:
- Core infrastructure subsystems that handle agent lifecycle, coordination, or interaction
- Protocol layer implementations (MCP capabilities) that expose agent functionality
- State management systems (revert engine) that track or restore agent state
- Configuration and selection logic that operates on agent abstractions
- Any module that needs to reference agent contracts without instantiating concrete implementations

Out of scope:
- Concrete agent implementation classes (these implement the interface rather than importing it)
- Agent factory or instantiation logic that necessarily depends on concrete classes
- Test fixtures that deliberately instantiate specific agent implementations
- Migration or adapter code that bridges between different agent contract versions

## Rationale

- The pattern appears across 4 distinct infrastructure subsystems (MCP capabilities, revert engine, config utilities, agent selection) with 0.90 significance, indicating deliberate architectural consistency rather than coincidental imports.
- Centralizing the agent contract in a single interface module enables dependency inversion, allowing infrastructure to depend on stable abstractions while agent implementations remain flexible and independently evolvable.
- The relative import path ../agents/IAgent suggests a first-party module designed specifically as the project's agent contract boundary, distinguishing it from ad-hoc type definitions or external dependencies.
- Co-location of IAgent imports with other infrastructure modules (ConfigLoader, agent-utils, constants) demonstrates that the interface serves as a foundational contract alongside other core abstractions.

## Consequences

Positive:
- Infrastructure subsystems remain decoupled from concrete agent implementations, enabling independent evolution of agent classes without cascading changes to protocol handlers, state management, or configuration logic.
- Consistent contract boundary across all infrastructure reduces cognitive load and makes agent interaction patterns predictable for developers working in different subsystems.
- Dependency inversion prevents circular dependencies between core infrastructure and agent implementations, supporting cleaner module boundaries and testability.
- Polymorphic agent handling becomes straightforward as all infrastructure references the same interface contract, enabling uniform treatment of different agent types.

Negative:
- Changes to the IAgent interface contract require coordinated updates across all 4+ consuming infrastructure modules, creating a potential bottleneck for interface evolution.
- The centralized interface becomes a critical dependency point; breaking changes or versioning challenges affect multiple subsystems simultaneously.
- Developers must maintain discipline to import the interface rather than concrete classes, as the type system may not prevent direct imports of implementations in all contexts.
- Interface abstraction may introduce indirection that obscures which concrete agent capabilities are actually available at runtime, requiring additional documentation or runtime checks.

## Alternatives

- Define agent contracts locally within each infrastructure subsystem based on that subsystem's specific needs (rejected)
  Rejected because: Would lead to inconsistent agent abstractions across subsystems, making it difficult to pass agent references between protocol handlers, state management, and selection logic. The evidence shows a deliberate choice for a single shared interface.
  When valid: In projects with truly independent subsystems that never share agent references or where each subsystem requires fundamentally different agent contracts
- Import concrete agent classes directly and rely on structural typing or duck typing for polymorphism (rejected)
  Rejected because: Creates tight coupling between infrastructure and implementations, preventing independent evolution and introducing circular dependency risks. The observed pattern explicitly uses an interface module to avoid this coupling.
  When valid: In small projects with a single agent implementation where abstraction overhead outweighs decoupling benefits
- Use a protocol or trait-based approach with multiple interface modules for different agent capabilities (deferred)
  When valid: If agent capabilities become highly diverse and subsystems only need subsets of the full agent contract, splitting into capability-specific interfaces could reduce coupling. Not currently evidenced in the codebase.

## Risks

- Interface evolution becomes constrained by the need to maintain compatibility across 4+ infrastructure subsystems, potentially slowing agent capability development
  Mitigation: Establish interface versioning strategy and consider capability-based interface composition if the contract grows too large. Document breaking change procedures that account for all consuming subsystems.
  Owner: engineering team
- Developers may inadvertently import concrete agent classes instead of the interface, breaking the abstraction boundary and introducing coupling
  Mitigation: Implement linting rules or import analysis to detect direct imports of concrete agent classes in infrastructure code. Code review checklist should verify interface usage in new infrastructure modules.
  Owner: engineering team
- The interface abstraction may become a leaky abstraction if infrastructure code needs to check for specific agent implementation types at runtime
  Mitigation: Design the IAgent interface to expose capability queries or feature flags that allow infrastructure to check capabilities without type-checking concrete classes. Document when runtime type checks are acceptable exceptions.
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
- When implementing new infrastructure that interacts with agents, examine the import patterns in the 4 observed files (MCP capabilities, revert engine, config utilities, agent selection) to understand how the interface is used alongside other infrastructure modules.
- The interface module location uses a relative path pattern, suggesting it resides in a sibling directory to core infrastructure. Maintain this module organization when adding new infrastructure subsystems.
- If infrastructure code needs agent-specific behavior beyond the interface contract, consider extending the interface with capability queries rather than importing concrete classes, preserving the abstraction boundary.

## Continuation Context


Verify commands:
- Discover the project's module analysis or import checking tooling from the repository and execute it to verify that infrastructure modules import the interface module rather than concrete agent classes
- Locate the project's static analysis or linting configuration and run the configured checks to detect any violations of the interface import pattern in core infrastructure code
- Identify the project's test suite and execute tests covering the 4 observed infrastructure subsystems to confirm they interact with agents through the interface contract

Accept when:
- All core infrastructure modules (protocol handlers, state management, configuration, selection logic) import the interface module when referencing agent contracts
- No direct imports of concrete agent implementation classes exist in infrastructure code where only the interface contract is required
- Static analysis or linting checks pass, confirming adherence to the interface import pattern across all infrastructure subsystems

## Enforcement

- Verified by: Static analysis or linting rules that detect imports of concrete agent classes in infrastructure modules
- Verified by: Code review checklist verification that new infrastructure code imports the interface module rather than concrete implementations
- Verified by: Automated import analysis in continuous integration that flags violations of the interface abstraction boundary
- Violation handling: Pull requests introducing direct imports of concrete agent classes in infrastructure code are blocked until refactored to use the interface
- Violation handling: Existing violations discovered through static analysis are logged as technical debt items and prioritized for refactoring
- Violation handling: Violations that introduce circular dependencies or break module boundaries are treated as high-priority defects requiring immediate remediation
- Exception process: Exceptions require documented justification explaining why the interface abstraction is insufficient for the specific use case
- Exception process: Exception requests must propose either an interface extension to support the use case or demonstrate that the code is legitimately outside policy scope (e.g., agent factory logic)
- Exception process: Approved exceptions are recorded with expiration dates and reviewed during interface evolution planning to determine if the interface should be extended to eliminate the exception