# Core Infrastructure Depends on Agent Abstraction Interface and Centralized Configuration Loader: Core Infrastructure Components Depend Iagent Interface

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The system contains core infrastructure components responsible for agent orchestration (revert-engine) and agent selection logic that must coordinate multiple agent implementations
- To avoid tight coupling between core infrastructure and specific agent implementations, the architecture introduces an abstraction layer through the IAgent interface
- Configuration management is centralized through a ConfigLoader module to ensure consistent configuration access across core components
- Both revert-engine and agent-selection components share dependencies on these internal abstractions, establishing a common dependency pattern within the core layer
- The pattern enables core infrastructure to remain stable while agent implementations can vary independently

## Problem Statement

Core infrastructure components need to coordinate and select among multiple agent implementations without creating tight coupling to specific agent types. Without a shared abstraction layer and centralized configuration, core components would depend directly on concrete agent implementations, making the system brittle and difficult to extend with new agent types.

## Decision

1. MUST: Core infrastructure components MUST depend on the IAgent interface abstraction rather than concrete agent implementation classes

## Policy Block

- MUST Core infrastructure components MUST depend on the IAgent interface abstraction rather than concrete agent implementation classes

In scope:
- All modules within the core infrastructure directory that coordinate, select, or orchestrate agent behavior
- Components that require configuration data to make decisions about agent usage or behavior
- Infrastructure code that needs to interact with multiple agent types through a uniform interface

Out of scope:
- Agent implementation modules themselves, which may depend on concrete classes within their own boundaries
- Utility modules that do not coordinate agent behavior
- Presentation or API layer components that interact with core infrastructure through higher-level abstractions

## Rationale

- The evidence shows both revert-engine and agent-selection components importing IAgent and ConfigLoader, establishing a consistent dependency pattern across core infrastructure
- Depending on an interface abstraction (IAgent) rather than concrete implementations enables the system to add new agent types without modifying core infrastructure code
- Centralizing configuration loading through ConfigLoader ensures consistent configuration interpretation and reduces the risk of configuration-related bugs from duplicate loading logic
- The relative import structure (../ traversal) indicates an intentional module boundary between core infrastructure and agent abstractions, supporting architectural layering

## Consequences

Positive:
- Core infrastructure remains stable when new agent implementations are added, reducing regression risk
- Agent implementations can be developed, tested, and deployed independently as long as they satisfy the IAgent interface contract
- Configuration changes are handled consistently across all core components through the single ConfigLoader module
- The abstraction boundary enables easier unit testing of core components by allowing mock agent implementations

Negative:
- Introduces an additional layer of indirection that developers must understand when tracing code execution from core infrastructure to agent behavior
- Changes to the IAgent interface contract require coordinated updates across all agent implementations
- The centralized ConfigLoader becomes a critical dependency point; failures in configuration loading affect all core components
- Relative import paths with upward traversal can become fragile if the directory structure is reorganized

## Alternatives

- Direct dependency from core infrastructure to concrete agent implementation classes (rejected)
  Rejected because: Creates tight coupling between core infrastructure and specific agent implementations, making it difficult to add new agent types without modifying core code. Violates the Open-Closed Principle.
  When valid: Only appropriate in systems with a single, unchanging agent implementation where extensibility is not a requirement
- Dependency injection container to provide agent instances and configuration to core components (rejected)
  Rejected because: Evidence shows direct imports of IAgent and ConfigLoader rather than injection patterns. While DI would provide additional flexibility, the current pattern is simpler and sufficient for the observed use cases.
  When valid: Appropriate when core components need runtime-configurable dependencies or when testing requires frequent dependency substitution
- Distributed configuration with each core component loading its own configuration independently (rejected)
  Rejected because: Would lead to inconsistent configuration interpretation across core components and duplicate configuration loading logic. The centralized ConfigLoader ensures consistency.
  When valid: Only appropriate when different core components genuinely require isolated, non-overlapping configuration namespaces

## Risks

- The IAgent interface may become bloated over time as new agent capabilities are added, forcing all implementations to satisfy an increasingly complex contract
  Mitigation: Monitor interface growth and consider splitting into focused role interfaces if the contract becomes too broad. Apply Interface Segregation Principle when extending agent capabilities.
  Owner: Core infrastructure team
- Centralized ConfigLoader creates a single point of failure; configuration loading errors will cascade to all core components simultaneously
  Mitigation: Implement robust error handling and validation in ConfigLoader. Consider configuration caching and graceful degradation strategies for non-critical configuration values.
  Owner: Core infrastructure team
- Relative import paths with directory traversal become brittle if the project structure is refactored, potentially breaking all core component imports
  Mitigation: Consider introducing path aliases or module resolution configuration to make imports more resilient to directory restructuring. Document the module boundary conventions clearly.
  Owner: Engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When adding new core infrastructure components, examine existing core modules to identify the established import patterns for IAgent and ConfigLoader, then replicate those patterns to maintain consistency
- When implementing new agent types, ensure they fully satisfy the IAgent interface contract before integrating with core infrastructure to avoid runtime type errors
- When modifying the IAgent interface, audit all agent implementations and core infrastructure usage sites to ensure coordinated updates across the dependency boundary

## Continuation Context


Verify commands:
- Discover the project's module resolution configuration and verify that relative imports from core infrastructure to agent abstractions resolve correctly
- Locate and execute the project's static analysis or type checking tooling to verify that all core infrastructure imports of IAgent and ConfigLoader satisfy type contracts
- Identify the project's test suite and run tests covering core infrastructure components to verify that agent abstraction dependencies function correctly

Accept when:
- Static analysis confirms all core infrastructure modules import IAgent interface rather than concrete agent classes
- Type checking verifies that all uses of IAgent and ConfigLoader in core infrastructure satisfy their respective interface contracts
- Test suite passes for core infrastructure components, confirming that agent abstraction dependencies integrate correctly

## Enforcement

- Verified by: Static analysis tooling checks import statements in core infrastructure modules to ensure they reference agent abstractions rather than concrete implementations
- Verified by: Code review process verifies that new core infrastructure components follow the established dependency pattern on IAgent and ConfigLoader
- Verified by: Type checking in continuous integration pipeline catches violations of interface contracts at build time
- Violation handling: Build failures from type checking violations block merge until corrected
- Violation handling: Code review feedback requires refactoring of direct concrete agent dependencies to use IAgent abstraction
- Violation handling: Static analysis warnings for incorrect import patterns are escalated to errors in CI pipeline
- Exception process: Exceptions to the abstraction dependency rule require architectural review and explicit documentation of why direct concrete dependencies are necessary
- Exception process: Temporary exceptions during refactoring must include a remediation plan and timeline for returning to compliance
- Exception process: All exceptions must be documented in code comments with rationale and approval reference