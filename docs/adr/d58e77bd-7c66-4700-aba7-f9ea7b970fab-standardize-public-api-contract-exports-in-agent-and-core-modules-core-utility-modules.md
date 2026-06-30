# Standardize Public API Contract Exports in Agent and Core Modules: Core Utility Modules

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent implementations and core utility modules that expose public interfaces.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, ZedAgent) and core utility modules (SubagentsUtils, SubagentsProcessor) that expose public interfaces for external consumption
- Each module defines explicit public contracts through exported types, classes, and functions that serve as integration points for other system components
- The pattern emerged across 4 files with consistent naming conventions and export strategies, indicating an established architectural practice rather than ad-hoc implementation
- Agent modules import from core utilities (IAgent, FileSystemUtils) and expose their own contracts, creating a layered dependency structure with clear interface boundaries
- The presence of structured exports (ParsedFrontmatter, CopilotToolMapping, discoverSubagents) alongside implementation code suggests intentional separation between public API surface and internal logic

## Problem Statement

Without standardized public API contract definitions, modules risk exposing implementation details, creating tight coupling between components, and making it difficult for consumers to understand which interfaces are stable and supported versus internal and subject to change. This leads to fragile integrations and increased maintenance burden when refactoring internal implementations.

## Decision

1. MUST: Core utility modules MUST explicitly export all public types, interfaces, and functions intended for external consumption (e.g., ParsedFrontmatter, CopilotToolMapping, parseFrontmatter, validateFrontmatter)

## Policy Block

- MUST Core utility modules MUST explicitly export all public types, interfaces, and functions intended for external consumption (e.g., ParsedFrontmatter, CopilotToolMapping, parseFrontmatter, validateFrontmatter)

In scope:
- All TypeScript modules in src/agents/ directory
- All TypeScript modules in src/core/ directory that provide shared utilities
- Any module that exports types, classes, or functions consumed by other modules
- Interface definitions and type declarations intended for cross-module usage

Out of scope:
- Internal helper functions not exported from modules
- Private class methods and properties
- Test files and test utilities
- Build artifacts and generated code
- Third-party library exports (re-exports are in scope if intentional)

Exceptions:
- EXC-001: A module needs to temporarily expose internal implementation for debugging or migration purposes

## Rationale

- The evidence shows consistent export patterns across 4 files with 90.60% confidence, indicating this is an established architectural practice worth codifying
- Explicit public contracts enable loose coupling between agents and core utilities, allowing independent evolution of implementations while maintaining stable interfaces
- Named exports (OpenCodeAgent, ZedAgent, ParsedFrontmatter, etc.) provide clear entry points for module consumers and improve code discoverability
- The pattern supports extensibility by allowing new agent types to implement common interfaces (IAgent) while exposing their own specialized contracts

## Consequences

Positive:
- Clear API boundaries reduce coupling and make refactoring safer by limiting the surface area of breaking changes
- Explicit exports improve code comprehension for developers integrating with agents or core utilities
- TypeScript type checking provides compile-time verification of contract adherence across module boundaries
- Consistent export patterns enable automated tooling for API documentation generation and contract testing

Negative:
- Requires discipline to maintain separation between public and private APIs, adding cognitive overhead during development
- May lead to verbose export lists in modules with many public contracts
- Changes to public contracts require coordination with all consumers, potentially slowing feature development
- Risk of contract proliferation if not carefully managed, leading to API surface bloat

## Alternatives

- Export all module contents using wildcard exports (export *) (rejected)
  Rejected because: Wildcard exports expose implementation details, create unclear API boundaries, and make it difficult to track what is part of the public contract versus internal implementation
  When valid: Never recommended for production code; only acceptable in test utilities or internal-only modules
- Use default exports for primary module contracts instead of named exports (rejected)
  Rejected because: Default exports reduce discoverability, make refactoring harder (no consistent naming), and conflict with the observed pattern of named exports across the codebase
  When valid: Could be considered for modules with a single, obvious primary export, but evidence shows named exports are the established pattern
- Define all public contracts in a centralized types package (deferred)
  Rejected because: Not rejected, but not currently implemented; would require significant refactoring
  When valid: Valid for future consideration if contract sharing becomes more complex or if circular dependencies emerge

## Risks

- Developers may inadvertently export internal implementation details, weakening API boundaries over time
  Mitigation: Implement linting rules to flag unexpected exports and conduct code reviews focused on API surface area
  Owner: Engineering team
- Public contract changes may break downstream consumers without clear versioning or deprecation strategy
  Mitigation: Establish semantic versioning for API contracts and require deprecation warnings before removing public exports
  Owner: Architecture team
- Over-specification of contracts may create rigidity, making it difficult to evolve implementations
  Mitigation: Design contracts at appropriate abstraction levels, favoring behavioral interfaces over implementation-specific types
  Owner: Engineering team

## Implementation Notes

- Use TypeScript's export keyword explicitly for each public contract rather than collecting exports at the end of files
- Consider creating index.ts files in directories to aggregate and re-export public contracts from multiple modules
- Document public contracts with JSDoc comments explaining their purpose, usage, and any stability guarantees
- Use TypeScript's 'export type' for type-only exports to enable better tree-shaking and clarify intent
- Establish naming conventions for exported contracts (e.g., Agent suffix for agent classes, Mapping suffix for configuration types)

## Continuation Context


Verify commands:
- grep -r "^export " src/agents/ src/core/ | grep -v "export default" | wc -l
- npx ts-node -e "import * as agents from './src/agents/OpenCodeAgent'; console.log(Object.keys(agents))"
- find src/agents src/core -name '*.ts' -exec grep -L "^export " {} \; | grep -v ".test.ts" | grep -v ".spec.ts"

Accept when:
- All agent modules export their primary class or interface with a descriptive name
- Core utility modules explicitly list all public exports without wildcard patterns
- No internal helper functions or private implementation details appear in module export lists
- TypeScript compilation succeeds with strict mode enabled, verifying all contract types are valid

## Enforcement

- Verified by: TypeScript compiler checks during CI build process
- Verified by: Code review checklist item for API surface area review
- Verified by: Automated linting rules checking export patterns
- Verified by: API documentation generation that flags undocumented exports
- Violation handling: CI build fails if TypeScript compilation errors occur due to contract violations
- Violation handling: Code review requires explicit justification for any new public exports
- Violation handling: Linter warnings must be addressed before merge approval
- Violation handling: Quarterly API surface area audits to identify and remediate violations
- Exception process: Developer documents exception rationale in code comments and pull request description
- Exception process: Architecture team reviews exception request within 2 business days
- Exception process: Approved exceptions are tracked in architecture decision log with expiration dates
- Exception process: Exceptions are revisited during quarterly architecture reviews for potential removal