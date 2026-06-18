# Standardize Public API Contracts Through Exported Interfaces and Types: Modules Locate Interface

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all modules and libraries that expose public APIs within the codebase.

## Context

- The codebase contains 25 modules that define and export public API contracts through TypeScript interfaces, types, and classes, establishing clear boundaries between implementation and consumer code.
- Agent implementations (GeminiCliAgent, JulesAgent, JunieAgent, KiroAgent, GooseAgent, KiloCodeAgent, JetBrainsAiAssistantAgent, AgentsMdAgent) consistently export their class definitions as public contracts, following a uniform pattern across the agent subsystem.
- Core utilities (UnifiedConfigLoader, GitignoreUtils, SkillsProcessor, RuleProcessor) expose typed interfaces (UnifiedLoadOptions, IAgentConfig, IAgent) and functions as explicit API boundaries, separating internal logic from external consumption.
- The pattern emerged from the need to support multiple agent implementations and configuration loaders while maintaining type safety and clear module boundaries in a TypeScript codebase.
- CLI handlers and revert utilities depend on these public contracts to coordinate cross-module operations without coupling to internal implementation details.

## Problem Statement

Without standardized public API contracts, modules risk exposing implementation details, creating tight coupling between components, and making it difficult to evolve internal logic without breaking consumers. The absence of explicit interface definitions leads to implicit dependencies, reduced testability, and unclear boundaries between public and private APIs.

## Decision

1. MAY: Modules MAY co-locate interface definitions with implementations when the interface is only consumed by a single client.

## Policy Block

- MAY Modules MAY co-locate interface definitions with implementations when the interface is only consumed by a single client.

In scope:
- All agent implementations in src/agents/
- Core utility modules in src/core/ that are imported by other modules
- CLI handlers in src/cli/ that coordinate cross-module operations
- Configuration loaders and processors
- Public functions exported from revert.ts and other top-level modules

Out of scope:
- Internal helper functions not exported from modules
- Private implementation details within classes
- Test utilities and fixtures
- Build and tooling scripts

Exceptions:
- EXC-001: A module is purely internal and never imported by other modules
- EXC-002: Rapid prototyping or experimental features where API stability is not yet required

## Rationale

- The evidence shows 25 files consistently exporting public contracts with 90.24% confidence, indicating this is an established architectural pattern rather than an isolated practice.
- TypeScript's type system provides compile-time verification of contract adherence, reducing runtime errors and improving refactoring safety across module boundaries.
- Explicit API contracts enable independent evolution of implementations (as seen with multiple agent types) while maintaining compatibility with consumers like CLI handlers and configuration loaders.
- The pattern supports testability by allowing mock implementations of interfaces (IAgent, IAgentConfig) without requiring concrete class dependencies.

## Consequences

Positive:
- Clear module boundaries reduce coupling and enable independent evolution of implementations without breaking consumers.
- TypeScript type checking catches contract violations at compile time, reducing runtime errors and improving developer confidence during refactoring.
- Explicit interfaces improve code discoverability and documentation, making it easier for developers to understand module capabilities and usage patterns.
- Testability improves through dependency injection and mock implementations based on interface contracts rather than concrete classes.

Negative:
- Additional boilerplate required to define and maintain interface definitions alongside implementations, increasing initial development time.
- Interface changes require coordinated updates across all implementations and consumers, potentially creating migration overhead.
- Over-abstraction risk if interfaces are defined prematurely before usage patterns are well understood, leading to unnecessary complexity.
- TypeScript compilation overhead increases with additional type definitions and interface checking across module boundaries.

## Alternatives

- Use implicit contracts based on structural typing without explicit interface definitions (rejected)
  Rejected because: Implicit contracts lack discoverability, make breaking changes harder to detect, and provide no compile-time guarantees about API stability across module boundaries.
  When valid: Only appropriate for small, single-developer projects where all code is maintained by one person with full context.
- Define all interfaces in a single central contracts file or package (rejected)
  Rejected because: Centralized contracts create a bottleneck for changes, increase merge conflicts, and couple unrelated modules through shared definition files.
  When valid: May be appropriate for stable, cross-cutting contracts like error types or logging interfaces that truly span the entire system.
- Use JSDoc type annotations instead of TypeScript interfaces (rejected)
  Rejected because: JSDoc provides weaker type checking, no compile-time enforcement, and less tooling support compared to native TypeScript interfaces.
  When valid: Appropriate for JavaScript-only codebases or when TypeScript adoption is not feasible due to build constraints.

## Risks

- Interface proliferation leads to over-abstraction and unnecessary complexity when interfaces are defined before usage patterns are clear.
  Mitigation: Defer interface extraction until at least two consumers need the contract, following the rule of three for abstraction. Review interfaces during architecture reviews to identify and remove unused abstractions.
  Owner: Engineering team with architecture review oversight
- Breaking changes to widely-used interfaces (like IAgent) require coordinated updates across many implementations, creating migration risk and potential system instability.
  Mitigation: Use semantic versioning for interface changes, provide deprecation warnings before breaking changes, and maintain backward compatibility adapters during transition periods. Document all interface changes in CHANGELOG.
  Owner: Module owners with tech lead approval for breaking changes
- Inconsistent interface naming and organization patterns across modules reduce discoverability and create confusion about which contracts are public vs internal.
  Mitigation: Establish naming conventions (IAgent, UnifiedLoadOptions) and file organization patterns (separate IAgent.ts for shared interfaces). Document conventions in architecture guidelines and enforce through code review.
  Owner: Engineering team with style guide enforcement in code review

## Implementation Notes

- Follow the established pattern of prefixing interface names with 'I' (IAgent, IAgentConfig) for interface definitions that define contracts, while using descriptive names for type aliases (UnifiedLoadOptions).
- Separate interface definitions into dedicated files (e.g., IAgent.ts) when the interface is implemented by multiple classes or consumed by multiple modules, as demonstrated by the IAgent interface pattern.
- Export public functions with explicit return type annotations, especially for async operations (loadUnifiedConfig, discoverSkills, applyRulerConfig) to ensure contract clarity.
- Co-locate related interfaces and types with their primary implementation when they form a cohesive module boundary, as seen with UnifiedConfigLoader and UnifiedLoadOptions.

## Continuation Context


Verify commands:
- grep -r "export.*interface" src/ --include="*.ts" | wc -l
- grep -r "export.*class.*Agent" src/agents/ --include="*.ts" | wc -l
- tsc --noEmit --strict && echo "Type checking passed"

Accept when:
- All modules in src/agents/ and src/core/ that are imported by other modules export at least one public interface, type, or class definition.
- TypeScript compilation succeeds with strict mode enabled, confirming all exported contracts are properly typed.
- Code review confirms that no internal implementation details are exposed through public exports, maintaining clear API boundaries.

## Enforcement

- Verified by: TypeScript compiler strict mode checks during CI build process
- Verified by: Code review checklist item verifying public API contracts are defined for new modules
- Verified by: Automated linting rules checking for exported members without type annotations
- Violation handling: CI build fails if TypeScript compilation errors occur due to missing or incorrect type definitions
- Violation handling: Code review blocks merge if new modules lack explicit public API contracts
- Violation handling: Architecture review required for any changes to widely-used interfaces like IAgent or IAgentConfig
- Exception process: Developer documents rationale for exception in module header comment or ADR
- Exception process: Tech lead or architect reviews and approves exception based on documented justification
- Exception process: Exception is time-bound with documented plan to bring module into compliance or remove it