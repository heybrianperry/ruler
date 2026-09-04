# Core Agent Modules Centralize Interface Contracts and Configuration Management: Core Components Not Duplicate Agent Interface

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The system implements multiple core engines (agent-selection, revert-engine) that require consistent access to agent interfaces and configuration
- Agent behavior contracts and configuration loading logic need to be shared across different core components to maintain consistency
- The codebase organizes internal modules using relative path imports, establishing a dependency structure where core components depend on shared agent modules
- Both selection and revert engines require the IAgent interface, indicating a common contract for agent implementations across different operational contexts

## Problem Statement

Core system components need consistent access to agent interface contracts, configuration management, and shared utilities without duplicating implementation logic across multiple engines and subsystems.

## Decision

1. MUST_NOT: Core components MUST NOT duplicate agent interface definitions or configuration loading logic that already exists in centralized modules

## Policy Block

- MUST_NOT Core components MUST NOT duplicate agent interface definitions or configuration loading logic that already exists in centralized modules

In scope:
- All modules within the core system directory that interact with agent implementations
- Components that require configuration loading or access to system constants
- Engine implementations that orchestrate agent behavior (selection, revert, apply)

Out of scope:
- Agent implementations themselves, which implement the IAgent interface rather than importing it for dependency
- External tooling or scripts that operate outside the core system architecture
- Test fixtures that may need to mock or stub agent interfaces independently

## Rationale

- Centralizing agent interface contracts ensures all core components operate against the same agent contract, preventing interface drift and integration issues
- Shared configuration loading through ConfigLoader eliminates duplicate configuration parsing logic and ensures consistent configuration interpretation across engines
- The pattern is evidenced by both agent-selection and revert-engine importing the same core modules, demonstrating actual reuse across different operational contexts
- Relative path imports make dependency relationships explicit and enable static analysis tools to verify module boundaries

## Consequences

Positive:
- Interface changes to IAgent automatically propagate to all core components, reducing maintenance burden
- Configuration loading behavior remains consistent across all engines and core components
- Code reuse reduces duplication and the risk of divergent implementations
- Explicit import paths make module dependencies traceable and verifiable through static analysis

Negative:
- Core components become tightly coupled to the centralized module structure, making it harder to extract or refactor individual components
- Changes to shared modules like IAgent or ConfigLoader have broad impact across multiple core components
- Relative path imports create maintenance overhead if directory structure changes
- New developers must understand the centralized module architecture before implementing core components

## Alternatives

- Allow each core component to define its own agent interfaces and configuration loading logic independently (rejected)
  Rejected because: Would lead to interface drift, duplicate configuration parsing logic, and inconsistent agent behavior across different engines
  When valid: Only appropriate for isolated components that genuinely require different agent contracts or configuration schemas
- Use a dependency injection container to provide agent interfaces and configuration at runtime rather than static imports (rejected)
  Rejected because: Adds runtime complexity and obscures compile-time dependency relationships that are currently explicit through imports
  When valid: May be appropriate if the system needs to support plugin architectures or runtime component swapping
- Package shared modules as internal workspace packages with explicit versioning rather than relative imports (deferred)
  Rejected because: Not rejected - could be a future evolution if the codebase grows to require stricter module boundaries and versioning
  When valid: Becomes valuable when the codebase scales to multiple teams or when shared modules need independent release cycles

## Risks

- Breaking changes to IAgent interface impact all core components simultaneously, potentially causing widespread failures
  Mitigation: Implement interface versioning or use TypeScript's type system to detect breaking changes at compile time before deployment
  Owner: engineering team
- Circular dependencies may emerge if core modules begin importing from components that depend on them
  Mitigation: Enforce unidirectional dependency flow through linting rules and architectural tests that detect circular imports
  Owner: engineering team
- Relative path imports become fragile if directory structure is reorganized during refactoring
  Mitigation: Use path aliases or module resolution configuration to decouple import paths from physical directory structure
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
- When implementing new core components, identify which shared modules are required by examining existing core components in the same directory
- Maintain the IAgent interface as the single source of truth for agent contracts - any new agent capabilities should be added to this interface rather than creating parallel interfaces
- If a core component requires configuration, verify that ConfigLoader supports the required configuration schema before implementing the component

## Continuation Context


Verify commands:
- Discover the project's module resolution configuration and verify that relative imports resolve correctly to the intended core modules
- Locate and execute the project's static analysis or linting tools to detect any circular dependencies between core modules
- Identify the project's type checking configuration and verify that all imports of IAgent and ConfigLoader satisfy the interface contracts

Accept when:
- All core components successfully import IAgent, ConfigLoader, and agent-utils without compilation errors
- Static analysis confirms no circular dependencies exist between core modules and their dependencies
- Type checking passes for all usages of imported interfaces and utilities, confirming contract compliance

## Enforcement

- Verified by: TypeScript compiler enforces interface contracts at build time
- Verified by: Static analysis tools detect circular dependencies and import violations during continuous integration
- Verified by: Code review verifies that new core components use centralized modules rather than duplicating functionality
- Violation handling: Build failures prevent deployment of code that violates interface contracts or creates circular dependencies
- Violation handling: Code review feedback requires refactoring of components that duplicate centralized module functionality
- Violation handling: Linting errors block pull request merges when import patterns violate established conventions
- Exception process: Exceptions require architectural review to determine if the component genuinely needs a different interface or if the centralized interface should be extended
- Exception process: Document approved exceptions in architecture decision records with clear justification for the deviation
- Exception process: Time-bound exceptions with a plan to either refactor the component or update the centralized modules to support the use case