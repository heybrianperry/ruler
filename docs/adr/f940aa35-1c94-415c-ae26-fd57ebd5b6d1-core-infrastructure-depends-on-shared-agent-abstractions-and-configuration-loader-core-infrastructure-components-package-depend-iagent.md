# Core Infrastructure Depends on Shared Agent Abstractions and Configuration Loader: Core Infrastructure Components Package Depend Iagent

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The core infrastructure layer coordinates agent behavior and configuration management across the system
- Multiple core components (revert-engine, agent-selection) require access to agent abstractions and configuration loading capabilities
- Shared internal modules (IAgent interface, ConfigLoader utility, agent-utils) provide foundational abstractions for agent coordination
- The pattern establishes a dependency relationship where core infrastructure components depend on these shared abstractions rather than implementing agent coordination logic independently

## Problem Statement

Core infrastructure components need consistent access to agent abstractions and configuration loading capabilities. Without shared internal modules, each component would implement its own agent coordination and configuration logic, leading to duplication, inconsistency, and tight coupling between infrastructure and agent implementation details.

## Decision

1. MUST: Core infrastructure components in the core package MUST depend on the IAgent interface for agent abstraction rather than concrete agent implementations

## Policy Block

- MUST Core infrastructure components in the core package MUST depend on the IAgent interface for agent abstraction rather than concrete agent implementations

In scope:
- All modules within the core infrastructure package that coordinate agent behavior
- All modules within the core infrastructure package that require configuration access
- Components implementing revert operations, agent selection, or other core orchestration logic

Out of scope:
- Agent implementation modules themselves (which implement IAgent rather than depend on it)
- Utility modules that do not coordinate agents or access configuration
- Presentation layer or API boundary components that interact with core infrastructure through higher-level interfaces

## Rationale

- The IAgent interface provides a stable abstraction boundary that decouples core infrastructure from concrete agent implementations, enabling agent implementations to evolve independently
- Centralizing configuration loading in ConfigLoader ensures consistent configuration access patterns and reduces duplication across core components
- Shared agent-utils module promotes code reuse and consistency in agent-related operations across the core infrastructure layer
- The pattern is evidenced by consistent imports of IAgent and ConfigLoader across multiple core infrastructure files (revert-engine.ts, agent-selection.ts)

## Consequences

Positive:
- Core infrastructure components are decoupled from concrete agent implementations through the IAgent interface abstraction
- Configuration access is centralized and consistent across core components via ConfigLoader
- Agent-related utility logic is shared and reusable rather than duplicated
- The dependency structure supports independent evolution of agent implementations and core infrastructure

Negative:
- Core infrastructure components have a mandatory dependency on the agents package for the IAgent interface
- Changes to IAgent interface or ConfigLoader API require coordination across multiple core infrastructure components
- Additional indirection layer between core infrastructure and agent implementations may complicate debugging
- New core infrastructure components must discover and adopt the shared abstraction pattern rather than implementing direct solutions

## Alternatives

- Implement agent coordination logic directly in each core infrastructure component without shared abstractions (rejected)
  Rejected because: Would lead to code duplication, inconsistent agent handling patterns, and tight coupling between infrastructure and agent implementation details
  When valid: Only appropriate for prototypes or single-component systems where abstraction overhead exceeds duplication cost
- Use dependency injection container to provide agent and configuration dependencies to core components (rejected)
  Rejected because: Evidence shows direct module imports rather than DI container usage; would require significant architectural refactoring
  When valid: Appropriate for larger systems requiring runtime dependency configuration or extensive testing with mock implementations
- Merge core infrastructure and agent modules into a single package to eliminate cross-package dependencies (rejected)
  Rejected because: Would violate separation of concerns between infrastructure orchestration and agent implementation; reduces modularity
  When valid: Appropriate only if the system is small enough that package boundaries provide no architectural value

## Risks

- Breaking changes to IAgent interface or ConfigLoader API impact multiple core infrastructure components simultaneously
  Mitigation: Maintain interface stability through versioning; use deprecation periods for breaking changes; ensure comprehensive test coverage of interface contracts
  Owner: engineering team
- Circular dependency risk if agent implementations attempt to import core infrastructure components
  Mitigation: Enforce unidirectional dependency flow: core depends on agent abstractions, agents implement abstractions but do not depend on core; use static analysis to detect circular imports
  Owner: engineering team
- New developers may bypass shared abstractions and implement direct agent coupling in new core components
  Mitigation: Document the dependency pattern in onboarding materials; use code review to enforce abstraction usage; consider linting rules to detect direct agent implementation imports in core package
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
- When creating new core infrastructure components, examine existing core modules to identify the import patterns for IAgent, ConfigLoader, and agent-utils
- Use relative path imports to reference shared internal modules; the import paths encode the package structure and dependency relationships
- If a core component requires agent coordination capabilities not provided by IAgent interface, extend the interface rather than bypassing it

## Continuation Context


Verify commands:
- Discover and execute the project's static analysis tooling to verify that core package modules import IAgent interface and ConfigLoader utility
- Discover and execute the project's dependency analysis tooling to confirm unidirectional dependency flow from core to agents package
- Discover and execute the project's test suite to verify that core infrastructure components correctly use IAgent abstraction and ConfigLoader utility

Accept when:
- Static analysis confirms all core infrastructure modules import IAgent and ConfigLoader from their respective shared module locations
- Dependency analysis confirms no circular dependencies between core and agents packages
- Test suite passes with core infrastructure components successfully coordinating agents through IAgent interface and accessing configuration through ConfigLoader

## Enforcement

- Verified by: Code review process verifies new core infrastructure components use shared abstractions
- Verified by: Static analysis in continuous integration detects direct imports of concrete agent implementations in core package
- Verified by: Dependency graph analysis detects circular dependencies or violations of unidirectional dependency flow
- Violation handling: Pull requests introducing direct agent implementation coupling in core components are rejected in code review
- Violation handling: Continuous integration fails if static analysis detects prohibited import patterns
- Violation handling: Violations discovered post-merge are tracked as technical debt items and prioritized for refactoring
- Exception process: Exception requests must document why shared abstractions are insufficient for the specific use case
- Exception process: Exceptions require approval from architecture review team or tech lead
- Exception process: Approved exceptions are documented as inline code comments with rationale and expiration conditions