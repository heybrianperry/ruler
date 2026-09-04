# Agent Base Class Architecture with AgentsMdAgent: Agent Implementations Import Additional Third Party

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent implementations in the codebase.

## Context

- The codebase contains multiple agent implementations (9 files) that require consistent behavior, interface contracts, and configuration file access patterns
- Agent implementations need to read configuration files in various formats (JSON, TOML) from the filesystem, requiring standardized file access utilities
- A common base class architecture (AgentsMdAgent, IAgent) has been established to provide shared functionality and enforce consistent agent behavior across all implementations
- Centralized filesystem utilities (FileSystemUtils) are provided to standardize configuration file discovery, reading, and parsing operations
- The pattern is observed across all agent types including QwenCodeAgent, RooCodeAgent, MistralVibeAgent, OpenCodeAgent, PiAgent, AiderAgent, AmpAgent, JulesAgent, and KiloCodeAgent

## Problem Statement

Without a common base class architecture and centralized utilities, agent implementations would duplicate configuration file access logic, diverge in their interface contracts, and create maintenance burden when behavioral changes need to propagate across all agent types. The system needs a standardized foundation that ensures all agents implement required interfaces while providing shared utilities for common operations like filesystem access.

## Decision

1. MAY: Agent implementations MAY import additional third-party parsing libraries for specific configuration formats when the base utilities do not provide the required functionality

## Policy Block

- MAY Agent implementations MAY import additional third-party parsing libraries for specific configuration formats when the base utilities do not provide the required functionality

In scope:
- All new agent implementations added to the agents directory
- Modifications to existing agent classes that handle configuration file access
- Refactoring of agent implementations to standardize behavior
- Agent implementations that need to read configuration files from the filesystem

Out of scope:
- Utility modules and helper functions that are not agent implementations
- Core framework modules that provide base functionality
- Test fixtures and mock agent implementations used solely for testing
- One-off scripts or tools that do not participate in the agent architecture

## Rationale

- The evidence shows 9 agent implementation files consistently importing and using AgentsMdAgent or IAgent base classes, demonstrating an established architectural pattern for agent inheritance
- Centralized FileSystemUtils module is imported by 4 agent files, indicating a deliberate choice to consolidate filesystem operations rather than allowing each agent to implement its own file access logic
- The pattern reduces code duplication across agent implementations and provides a single point of control for modifying agent behavior or filesystem access patterns
- Consistent base class usage ensures all agents implement required interface methods and can be used interchangeably where the base interface is expected

## Consequences

Positive:
- Reduced code duplication across agent implementations through shared base class functionality and centralized utilities
- Consistent interface contracts ensure all agents can be used interchangeably in contexts expecting the base interface
- Centralized filesystem utilities provide a single point of maintenance for configuration file access logic
- New agent implementations benefit from established patterns and utilities, reducing implementation time and errors

Negative:
- Agent implementations are tightly coupled to the base class architecture, making it difficult to implement agents with fundamentally different behavior patterns
- Changes to the base class or FileSystemUtils module may require updates across all 9+ agent implementations
- The inheritance hierarchy may become a bottleneck if base class modifications require extensive testing across all derived agents
- Developers must understand the base class contract and utility module APIs before implementing new agents

## Alternatives

- Allow each agent implementation to directly use Node.js filesystem modules without centralized utilities (rejected)
  Rejected because: This approach would lead to duplicated filesystem access logic across all agent implementations and make it difficult to standardize error handling, path resolution, and configuration file discovery patterns
  When valid: For one-off scripts or utilities that do not participate in the agent architecture and have unique filesystem access requirements
- Use composition over inheritance by providing utility functions without requiring a base class (rejected)
  Rejected because: Pure composition would not enforce interface contracts across agent implementations, making it difficult to ensure all agents implement required methods and can be used interchangeably
  When valid: For loosely coupled systems where agents do not need to share a common interface or be used polymorphically
- Define only an interface contract without providing a base class implementation (deferred)
  Rejected because: This approach would require each agent to reimplement common functionality, though it would provide more flexibility for agents with unique requirements
  When valid: When agent implementations have significantly different internal architectures but still need to satisfy a common interface contract

## Risks

- Base class modifications may introduce breaking changes across all agent implementations, requiring coordinated updates to 9+ files
  Mitigation: Implement comprehensive test coverage for the base class and all agent implementations; use semantic versioning for base class changes; provide deprecation warnings before removing base class functionality
  Owner: engineering team
- Tight coupling to FileSystemUtils may make it difficult to support alternative configuration sources (databases, remote APIs, environment variables)
  Mitigation: Design FileSystemUtils with an abstraction layer that can support multiple configuration sources; consider introducing a configuration provider interface if alternative sources are needed
  Owner: engineering team
- New developers may bypass the base class architecture if the pattern is not clearly documented or enforced through code review
  Mitigation: Add linting rules to detect agent implementations that do not extend the base class; document the architecture in developer onboarding materials; enforce pattern compliance during code review
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
- When implementing a new agent, start by examining the base class implementation to understand the required interface methods and available utility functions before writing agent-specific logic
- Use the FileSystemUtils module for all configuration file operations including path resolution, file existence checks, and content reading to maintain consistency with existing agent implementations
- Follow the established pattern of importing the base class and utilities using relative imports to maintain module cohesion and avoid circular dependencies

## Continuation Context


Verify commands:
- Discover the project's static analysis configuration and execute the linting rules that verify agent implementations extend the required base class
- Locate the project's test suite and run tests that validate agent interface compliance and base class behavior
- Search the agents directory for files matching the agent naming pattern and verify each imports the base class or interface

Accept when:
- All agent implementation files in the agents directory import and extend either AgentsMdAgent or implement IAgent interface
- Static analysis confirms no direct filesystem module imports in agent implementations where FileSystemUtils should be used
- All agent implementations pass interface compliance tests demonstrating they implement required base class methods

## Enforcement

- Verified by: Code review process verifies new agent implementations extend the base class and use centralized utilities
- Verified by: Static analysis and linting rules detect agent files that do not import the required base class
- Verified by: Automated tests validate that all agent implementations satisfy the base interface contract
- Violation handling: Pull requests introducing agent implementations that do not extend the base class are rejected during code review
- Violation handling: Linting failures block continuous integration pipeline execution until base class imports are added
- Violation handling: Existing violations are tracked as technical debt items and prioritized for refactoring
- Exception process: Exceptions require architectural review and approval from the engineering team lead
- Exception process: Exception requests must document why the base class architecture is incompatible with the specific agent requirements
- Exception process: Approved exceptions are documented in code comments with references to the exception approval decision