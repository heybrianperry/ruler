# Standardize TypeScript Type Definitions for Public API Contracts: Public Type Definitions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all TypeScript modules that define public API contracts, agent interfaces, configuration types, and MCP capabilities.

## Context

- The codebase contains multiple agent implementations (FactoryDroidAgent, CopilotAgent, AntigravityAgent, JulesAgent, AiderAgent, TraeAgent, ClineAgent, JetBrainsAiAssistantAgent, WarpAgent) that require consistent type definitions for interoperability
- Public API contracts are defined in core type files (src/types.ts, src/core/UnifiedConfigTypes.ts) that serve as the foundation for external integrations and agent communication
- MCP (Model Context Protocol) capabilities and agent utilities require strongly-typed interfaces to ensure type safety across the system
- The pattern appears in 43 files with 89.53% confidence, indicating a well-established architectural practice for managing public API surface area
- Configuration utilities and agent utilities depend on centralized type definitions to maintain consistency across different agent implementations

## Problem Statement

Without standardized TypeScript type definitions for public API contracts, the system faces risks of type inconsistencies across agent implementations, breaking changes in external integrations, and reduced maintainability as the API surface area grows. A consistent approach to defining and managing public API types is needed to ensure type safety, enable reliable agent interoperability, and provide clear contracts for external consumers.

## Decision

1. SHOULD: Public API type definitions SHOULD include JSDoc comments documenting the purpose, usage constraints, and examples for complex types

## Policy Block

- SHOULD Public API type definitions SHOULD include JSDoc comments documenting the purpose, usage constraints, and examples for complex types

In scope:
- All TypeScript files in src/types.ts and src/core/UnifiedConfigTypes.ts
- All agent implementation files (src/agents/*.ts)
- MCP capability definitions (src/mcp/capabilities.ts)
- Agent utility modules (src/agents/agent-utils.ts)
- Configuration utility modules (src/core/config-utils.ts)
- Any module that defines interfaces or types consumed by external integrations

Out of scope:
- Internal implementation details within a single module that are not exposed
- Test-specific type definitions used only in test files
- Third-party library type definitions from node_modules
- Build configuration types in jest.config.js or similar tooling files

Exceptions:
- EXC-001: An agent implementation requires a specialized type that is specific to that agent's unique functionality and will not be reused by other agents
- EXC-002: Rapid prototyping or experimental features are being developed that may not stabilize into the public API

## Rationale

- The pattern signature (dc1b57bd689dc0475c8072de90be279a) appears in 43 files with 89.53% confidence, demonstrating a strong, consistent architectural practice across the codebase
- Centralized type definitions in src/types.ts and src/core/UnifiedConfigTypes.ts enable multiple agent implementations to share common contracts, reducing duplication and ensuring interoperability
- TypeScript's type system provides compile-time guarantees that prevent type mismatches between agents and the core system, catching integration errors early in development
- Standardized public API types facilitate external integrations by providing a stable, well-documented contract that consumers can depend on across versions

## Consequences

Positive:
- Strong type safety across all agent implementations prevents runtime type errors and improves code reliability
- Centralized type definitions reduce code duplication and make it easier to evolve the API surface consistently
- External integrations benefit from clear, stable contracts that are less likely to break unexpectedly
- IDE tooling provides better autocomplete, refactoring support, and inline documentation for developers working with public APIs
- New agent implementations can quickly adopt existing type contracts, accelerating development time

Negative:
- Changes to core type definitions require careful coordination to avoid breaking multiple agent implementations simultaneously
- Developers must maintain discipline to use centralized types rather than creating ad-hoc interfaces, which requires code review enforcement
- Large type definition files (like UnifiedConfigTypes.ts) can become difficult to navigate as the API surface grows
- Overly strict typing may occasionally require workarounds or type assertions when dealing with dynamic or legacy code

## Alternatives

- Allow each agent to define its own types independently without shared contracts (rejected)
  Rejected because: This approach leads to type fragmentation, prevents agent interoperability, and makes it impossible to guarantee consistent behavior across the system. The evidence shows 43 files already following a centralized pattern, indicating this alternative was implicitly rejected.
  When valid: Only valid for completely isolated agents that never interact with the core system or other agents
- Use runtime validation with libraries like Zod or io-ts instead of compile-time TypeScript types (rejected)
  Rejected because: While runtime validation is valuable for external inputs, it does not replace the compile-time safety and IDE support that TypeScript types provide. Runtime validation should complement, not replace, static types.
  When valid: Should be used in addition to TypeScript types for validating untrusted external inputs at system boundaries
- Generate types automatically from OpenAPI/JSON Schema specifications (deferred)
  Rejected because: Not rejected, but deferred pending evaluation of tooling maturity and integration complexity
  When valid: Could be adopted in the future if the API surface becomes primarily REST-based and OpenAPI specifications are maintained as the source of truth

## Risks

- Core type definition files become bottlenecks as multiple teams need to modify them simultaneously, leading to merge conflicts and coordination overhead
  Mitigation: Implement a clear ownership model for type definitions, use TypeScript's module augmentation for extensions, and consider splitting large type files by domain when they exceed 500 lines
  Owner: Engineering team / API working group
- Breaking changes to public API types accidentally propagate to external consumers without proper versioning or deprecation warnings
  Mitigation: Implement automated type compatibility checking in CI, require explicit approval for changes to public API types, and maintain a changelog for type definition changes
  Owner: Engineering team / Release management
- Developers bypass centralized types by using 'any' or type assertions, undermining type safety guarantees
  Mitigation: Enable strict TypeScript compiler options (noImplicitAny, strict), enforce linting rules that flag excessive use of 'any', and review type assertions during code review
  Owner: Engineering team / Code reviewers

## Implementation Notes

- Start by auditing existing type definitions in src/types.ts and src/core/UnifiedConfigTypes.ts to ensure they are properly documented and exported
- Configure TypeScript compiler with strict mode and enable 'declaration: true' to generate .d.ts files for external consumers
- Establish a naming convention for public API types (e.g., prefix with 'Public' or suffix with 'Contract') to distinguish them from internal types
- Create a type definition review checklist that includes: JSDoc documentation, backward compatibility check, export verification, and usage examples
- Consider using TypeScript's 'satisfies' operator to validate that agent implementations conform to expected types without widening inference
- Set up automated tooling (e.g., api-extractor or TypeDoc) to generate API documentation from type definitions

## Continuation Context


Verify commands:
- grep -r "export \(interface\|type\)" src/types.ts src/core/UnifiedConfigTypes.ts | wc -l
- npx tsc --noEmit --strict && echo 'Type checking passed'
- grep -r ":\s*any" src/agents/ src/mcp/ | grep -v "node_modules" | wc -l

Accept when:
- All agent implementation files successfully compile with TypeScript strict mode enabled and reference types from src/types.ts or src/core/UnifiedConfigTypes.ts
- Public API type definition files (src/types.ts, src/core/UnifiedConfigTypes.ts) contain exported interfaces with JSDoc documentation
- The usage of 'any' type in agent implementations and MCP capabilities is minimal (fewer than 5 occurrences) and justified with comments
- CI pipeline includes type checking that fails on type errors or missing type definitions

## Enforcement

- Verified by: TypeScript compiler in strict mode as part of CI/CD pipeline
- Verified by: ESLint rules configured to flag usage of 'any' type and missing return types
- Verified by: Code review checklist requiring verification that new agent implementations use centralized type definitions
- Verified by: Automated API compatibility checks using tools like api-extractor or ts-morph to detect breaking changes
- Violation handling: CI build fails if TypeScript compilation errors occur or strict mode violations are detected
- Violation handling: Pull requests that introduce 'any' types or bypass centralized definitions are flagged for review and require explicit justification
- Violation handling: Breaking changes to public API types trigger automated notifications to API consumers and require migration guide documentation
- Violation handling: Quarterly audits of type definition usage to identify and refactor non-compliant code
- Exception process: Developer submits exception request with justification explaining why centralized types cannot be used
- Exception process: Tech lead or API working group reviews the request and assesses impact on type safety and maintainability
- Exception process: If approved, exception is documented in code with @exception JSDoc tag and tracked in a central exceptions registry
- Exception process: Exceptions are reviewed quarterly to determine if they can be resolved by evolving the centralized type definitions