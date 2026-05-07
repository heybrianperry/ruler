# Standardize TypeScript Module Exports for Public API Contracts: Public Modules Maintain

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all TypeScript modules that define public API contracts, interfaces, and utility functions exposed to external consumers or other system components.

## Context

- The codebase contains multiple TypeScript modules (SkillsUtils.ts, GitignoreUtils.ts, agent-selection.ts, IAgent.ts, etc.) that expose public APIs through consistent export patterns
- These modules serve as integration points between different system components, including agent implementations (JulesAgent, KiroAgent, OpenHandsAgent) and core processors (RuleProcessor, SkillsProcessor)
- A consistent pattern has emerged across 11 files with 89.09% confidence, indicating an established architectural convention for structuring public API contracts
- The pattern includes interface definitions, utility functions, and type definitions that form the contract layer between internal implementations and external consumers
- Jest configuration (jest.config.js) and unified configuration types (UnifiedConfigTypes.ts) demonstrate the need for stable, well-defined API boundaries in both runtime and testing contexts

## Problem Statement

Without standardized conventions for defining and exposing public API contracts in TypeScript modules, the codebase risks inconsistent interface definitions, unclear module boundaries, and fragile integration points that make it difficult for consumers to reliably interact with system components. This leads to increased coupling, reduced maintainability, and potential breaking changes when internal implementations evolve.

## Decision

1. MUST: Public API modules MUST maintain backward compatibility for exported interfaces and types unless a major version change is explicitly communicated

## Policy Block

- MUST Public API modules MUST maintain backward compatibility for exported interfaces and types unless a major version change is explicitly communicated

In scope:
- All TypeScript modules in src/core/ that expose utility functions or processing logic
- All agent interface definitions and implementations in src/agents/
- Configuration type definitions that are consumed by multiple modules
- Public API contracts used for inter-module communication
- Test configuration files that define public testing interfaces

Out of scope:
- Internal implementation details within private class methods
- Temporary variables and helper functions not intended for export
- Third-party library type definitions managed by external packages
- Build artifacts and compiled JavaScript output
- Development-only utilities not used in production code

Exceptions:
- EX-001: Legacy modules undergoing gradual migration to the new pattern
- EX-002: Experimental features in feature-flagged code paths

## Rationale

- The pattern detected across 11 files with 89.09% confidence indicates this is an established and successful architectural convention in the codebase
- Consistent export patterns for interfaces (IAgent.ts), utilities (SkillsUtils.ts, GitignoreUtils.ts), and processors (RuleProcessor.ts, SkillsProcessor.ts) demonstrate proven value in maintaining clear API boundaries
- TypeScript's type system provides compile-time guarantees for API contracts, reducing runtime errors and improving developer experience through IDE autocomplete and type checking
- Standardizing public API patterns enables better modularity, easier testing (as evidenced by jest.config.js), and clearer separation of concerns between agent implementations and core logic

## Consequences

Positive:
- Improved code maintainability through clear separation between public contracts and internal implementations
- Enhanced developer experience with consistent patterns across modules, reducing cognitive load when working with different components
- Better type safety and compile-time error detection through explicit interface definitions
- Easier testing and mocking due to well-defined interfaces and pure utility functions
- Reduced coupling between modules, enabling independent evolution of implementations while maintaining stable contracts

Negative:
- Additional upfront effort required to design and document public API contracts before implementation
- Potential for over-engineering simple utilities that don't require formal interface definitions
- Increased file count and project structure complexity with dedicated interface files
- Backward compatibility constraints may slow down refactoring of established APIs

## Alternatives

- Use default exports exclusively for all modules to simplify import statements (rejected)
  Rejected because: Default exports reduce clarity, make refactoring harder, and prevent tree-shaking optimization. Named exports provide better IDE support and explicit contracts.
  When valid: Only for single-purpose utility modules with one primary export
- Co-locate all interfaces and implementations in single files without separation (rejected)
  Rejected because: Mixing interfaces with implementations increases coupling and makes it harder to maintain clear API boundaries. Separate interface files enable better dependency management.
  When valid: For very small, self-contained modules with no external consumers
- Use runtime validation libraries (e.g., Zod, io-ts) for all public API boundaries (deferred)
  Rejected because: While runtime validation adds safety, it introduces performance overhead and additional dependencies. Consider for critical external API boundaries.
  When valid: For APIs that accept untrusted external input or cross network boundaries

## Risks

- Inconsistent adoption across the codebase leading to mixed patterns and confusion
  Mitigation: Implement linting rules to enforce export patterns and conduct code review training on the standard
  Owner: Engineering team leads
- Breaking changes to public APIs causing downstream integration failures
  Mitigation: Implement API versioning strategy, maintain changelog, and use deprecation warnings before removing exports
  Owner: API governance team
- Over-abstraction leading to unnecessary complexity for simple utilities
  Mitigation: Apply pattern pragmatically based on module scope and consumer count; allow exceptions for trivial internal utilities
  Owner: Architecture review board

## Implementation Notes

- Start by auditing existing modules to identify which exports are truly public vs. internal, then refactor to align with the standard
- Create a style guide documenting naming conventions for interface files (I-prefix), utility modules (-Utils suffix), and processor modules (-Processor suffix)
- Configure ESLint rules to enforce explicit return types on exported functions and prevent export of internal implementation details
- Establish a module index pattern (index.ts) for packages that need to re-export multiple public APIs from a single entry point
- Document all public APIs with TSDoc comments including @public tags, usage examples, and version information

## Continuation Context


Verify commands:
- grep -r "export default" src/ --include="*.ts" | grep -v "test" | wc -l
- find src/agents -name "*.ts" -exec grep -L "export.*interface" {} \;
- npx tsc --noEmit && echo "Type checking passed"
- grep -r "export.*{" src/core/ --include="*.ts" | head -20

Accept when:
- All agent implementations export interfaces from dedicated I*.ts files and conform to those interfaces
- Core utility modules (SkillsUtils, GitignoreUtils, etc.) use explicit named exports with full type signatures
- TypeScript compilation succeeds without errors and all public exports have documented types
- No internal implementation details (private helpers, temporary types) are exported from public API modules

## Enforcement

- Verified by: ESLint rules checking for explicit-module-boundary-types and no-default-export in public API modules
- Verified by: TypeScript compiler strict mode enabled with noImplicitAny and strictNullChecks
- Verified by: Code review checklist requiring verification of public API contracts before merge
- Verified by: Automated CI pipeline running type checking and linting on all pull requests
- Violation handling: CI build fails if TypeScript compilation errors occur or ESLint rules are violated
- Violation handling: Pull requests are blocked from merging until public API export patterns are corrected
- Violation handling: Automated comments on PRs highlight specific violations with links to this ADR and remediation guidance
- Violation handling: Quarterly audits identify non-compliant modules for prioritized refactoring
- Exception process: Developer submits exception request via architecture review board with justification and impact analysis
- Exception process: Exception requires approval from at least two senior engineers and documentation in module README
- Exception process: Approved exceptions are tracked in a central registry with review dates and migration plans
- Exception process: Temporary exceptions expire after 6 months and require renewal with progress update