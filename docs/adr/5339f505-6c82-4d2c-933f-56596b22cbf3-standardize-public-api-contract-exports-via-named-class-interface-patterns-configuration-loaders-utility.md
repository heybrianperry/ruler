# Standardize Public API Contract Exports via Named Class/Interface Patterns: Configuration Loaders Utility

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all modules that export public API contracts.

## Context

- The codebase contains multiple agent implementations (FirebenderAgent, MistralVibeAgent, GeminiCliAgent, AgentsMdAgent) and configuration utilities (UnifiedConfigLoader, ConfigLoader) that require consistent public interfaces for consumption by other modules.
- TypeScript modules across src/agents/, src/core/, src/paths/, and src/vscode/ export named classes, interfaces, and type definitions as their primary public contracts, establishing a pattern of explicit API boundaries.
- Configuration loading, file system operations, and agent management require stable, discoverable interfaces to support both internal coordination and potential external extension points.
- The pattern emerged organically across 14 files with 90.42% confidence, indicating a consistent architectural approach to module boundaries and contract definition rather than ad-hoc exports.

## Problem Statement

Without standardized public API contract patterns, modules risk exposing implementation details, creating tight coupling between components, and making it difficult for consumers to discover and correctly use exported functionality. Inconsistent export strategies lead to maintenance burden, refactoring friction, and unclear module boundaries.

## Decision

1. MUST: Configuration loaders and utility modules MUST export named interfaces or type definitions (e.g., UnifiedLoadOptions, VSCodeSettings, LoadedConfig) alongside their implementation functions.

## Policy Block

- MUST Configuration loaders and utility modules MUST export named interfaces or type definitions (e.g., UnifiedLoadOptions, VSCodeSettings, LoadedConfig) alongside their implementation functions.

In scope:
- All TypeScript modules in src/agents/ that implement agent behavior
- All TypeScript modules in src/core/ that provide configuration loading, file system utilities, or skill processing
- All TypeScript modules in src/paths/ and src/vscode/ that manage external integrations
- Any new modules that provide reusable functionality to other parts of the codebase

Out of scope:
- Internal implementation files that are not intended for consumption outside their immediate directory
- Test files and test utilities
- Build scripts and tooling configuration
- Private helper modules marked with underscore prefixes or explicitly documented as internal

Exceptions:
- EXC-001: A module provides a single, simple utility function with no associated types or configuration

## Rationale

- The pattern appears consistently across 14 files with 90.42% confidence, demonstrating an established architectural convention that has proven effective for module organization.
- Named exports for classes (FirebenderAgent, MistralVibeAgent, GeminiCliAgent, AgentsMdAgent) and interfaces (UnifiedLoadOptions, VSCodeSettings, LoadedConfig) provide clear, discoverable API boundaries that support both IDE tooling and human comprehension.
- Explicit contract definitions enable safe refactoring, as consumers depend on stable interface signatures rather than implementation details, reducing coupling and improving maintainability.
- The pattern aligns with TypeScript best practices for library and module design, leveraging the type system to enforce contracts at compile time.

## Consequences

Positive:
- Clear, discoverable API boundaries improve developer experience and reduce time spent understanding module interfaces
- Named exports enable better IDE support for auto-completion, refactoring, and type checking
- Explicit contracts reduce coupling between modules and enable independent evolution of implementations
- Consistent export patterns across the codebase reduce cognitive load and establish predictable conventions

Negative:
- Requires discipline to maintain clear boundaries between public contracts and internal implementation details
- May lead to more verbose import statements when consuming multiple exports from a single module
- Refactoring public contracts requires coordinated updates across all consumers, increasing migration effort
- Over-specification of types and interfaces can create maintenance burden if contracts are too granular

## Alternatives

- Use default exports for all modules with a single primary export (rejected)
  Rejected because: Default exports reduce discoverability, complicate refactoring, and make it harder to export multiple related contracts from a single module. The evidence shows consistent use of named exports across 14 files.
  When valid: Only for modules with a single, simple utility function and no associated types
- Export all functions and classes without explicit interface definitions (rejected)
  Rejected because: Without explicit interfaces, consumers couple to implementation details rather than contracts, reducing flexibility and increasing refactoring cost. The evidence shows consistent use of interfaces like IAgent and type definitions like UnifiedLoadOptions.
  When valid: Never for public APIs; only for internal implementation modules
- Use namespace exports to group related functionality (deferred)
  Rejected because: Not rejected, but not currently used in the codebase. Could be considered for future refactoring to group related utilities.
  When valid: When a module provides multiple related utilities that benefit from namespace grouping

## Risks

- Inconsistent application of the pattern across new modules as the codebase grows
  Mitigation: Establish linting rules to enforce named exports and document the pattern in contribution guidelines. Use code review to verify compliance.
  Owner: Engineering team
- Over-engineering of simple utility modules with unnecessary interface abstractions
  Mitigation: Provide clear guidance on when explicit interfaces are required (multi-implementation scenarios, external APIs) versus when simple function exports suffice.
  Owner: Tech lead
- Breaking changes to public contracts impact multiple consumers simultaneously
  Mitigation: Use semantic versioning, deprecation warnings, and migration guides when evolving public APIs. Consider maintaining backward compatibility layers for major changes.
  Owner: Engineering team

## Implementation Notes

- When creating a new agent, define a class that implements the IAgent interface and export both the class and any agent-specific configuration types (see FirebenderAgent, MistralVibeAgent as examples).
- For configuration utilities, export both the loading function and associated type definitions (e.g., loadConfig with LoadedConfig, ConfigOptions) to provide complete contract information to consumers.
- Use TypeScript's 'export { ... }' syntax at the end of files to create a clear manifest of public exports, making it easy to audit what constitutes the module's public API.
- Document public interfaces with JSDoc comments explaining their purpose, usage constraints, and any important behavioral guarantees.

## Continuation Context


Verify commands:
- grep -r 'export class\|export interface\|export type' src/agents/ src/core/ src/paths/ src/vscode/ | wc -l
- grep -r 'export default' src/agents/ src/core/ | grep -v '.test.ts' | wc -l
- npx ts-node -e "import * as agents from './src/agents'; console.log(Object.keys(agents).filter(k => k.includes('Agent')).length)"

Accept when:
- All agent modules export named classes implementing IAgent interface
- Configuration and utility modules export named interfaces/types alongside implementation functions
- Default exports are used only in documented exception cases (single utility functions)
- Public API exports are discoverable via IDE auto-completion and documented in module headers

## Enforcement

- Verified by: TypeScript compiler type checking during build
- Verified by: ESLint rules enforcing named exports over default exports
- Verified by: Code review checklist verifying public API contracts are documented
- Verified by: Automated tests importing and using public APIs to verify contract stability
- Violation handling: Build failures for modules that violate TypeScript interface contracts
- Violation handling: Linting warnings for default exports in public API modules
- Violation handling: Code review feedback requiring interface definitions for multi-implementation scenarios
- Violation handling: Refactoring tasks created for modules that expose implementation details
- Exception process: Developer documents rationale for exception in module header comment
- Exception process: Tech lead reviews and approves exception during code review
- Exception process: Exception is recorded in ADR exceptions log with justification
- Exception process: Exception is revisited during quarterly architecture review for potential refactoring