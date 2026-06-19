# Standardize TypeScript Module Exports for Public Agent APIs: Public Modules Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all TypeScript modules that expose public APIs, particularly agent implementations and core utilities that are consumed by external integrations or other modules within the system.

## Context

- The codebase contains multiple agent implementations (GooseAgent, FirebaseAgent, ClineAgent, KiroAgent, AugmentCodeAgent, AntigravityAgent, TraeAgent) that follow a consistent export pattern for public consumption
- Core utilities like GitignoreUtils, hash functions, and MCP merge functionality are designed to be consumed as stable public APIs by external integrations
- The pattern was detected across 14 files with 88.57% confidence, indicating a deliberate architectural choice for API contract stability
- TypeScript module exports serve as the primary contract boundary between internal implementation and external consumers
- Configuration files (jest.config.js, .prettierrc.js) and test harnesses demonstrate the need for predictable, well-defined module interfaces

## Problem Statement

Without standardized module export patterns for public APIs, external consumers face unpredictable interfaces, breaking changes become frequent, and the system lacks clear boundaries between public contracts and internal implementation details. This leads to tight coupling, difficult refactoring, and poor developer experience for API consumers.

## Decision

1. MUST: Public API modules MUST use TypeScript explicit export syntax (export { ... }) rather than implicit exports to clearly define the public contract

## Policy Block

- MUST Public API modules MUST use TypeScript explicit export syntax (export { ... }) rather than implicit exports to clearly define the public contract

In scope:
- All agent implementation files in src/agents/
- Core utility modules in src/core/ that are consumed externally
- MCP integration modules in src/mcp/
- Index files that aggregate and re-export public APIs
- Test harness modules that define public testing interfaces

Out of scope:
- Internal helper functions not intended for external use
- Private implementation details within agent classes
- Configuration files that are not imported as modules
- Build artifacts and generated code
- Development-only utilities and debugging tools

Exceptions:
- EXC-001: A module is explicitly marked as internal-only with a leading underscore (e.g., _internal.ts) and documented as not part of the public API
- EXC-002: Experimental features are being developed and need temporary export patterns before stabilization

## Rationale

- The detection of this pattern across 14 files with 88.57% confidence indicates a mature, intentional architectural decision that has proven effective
- Consistent export patterns reduce cognitive load for developers consuming these APIs and enable better tooling support (autocomplete, type checking, refactoring)
- Explicit public API boundaries enable safe internal refactoring without breaking external consumers, supporting long-term maintainability
- TypeScript's module system provides compile-time guarantees that export contracts are honored, preventing accidental API surface expansion

## Consequences

Positive:
- Clear contract boundaries between public APIs and internal implementation enable safe refactoring
- External consumers have predictable, stable interfaces that reduce integration friction
- TypeScript tooling provides excellent autocomplete and type safety for API consumers
- Tree-shaking and dead code elimination work effectively with explicit named exports
- Documentation generation tools can automatically identify and document public API surface

Negative:
- Developers must be disciplined about what they export, adding cognitive overhead during development
- Changing public APIs requires careful versioning and migration planning, slowing down some changes
- Index files that re-export APIs add an extra layer of indirection that must be maintained
- Strict export discipline may feel overly restrictive for rapid prototyping or experimental features

## Alternatives

- Use default exports exclusively for all modules to simplify import syntax (rejected)
  Rejected because: Default exports prevent tree-shaking, make refactoring harder, and don't compose well in index files that aggregate multiple exports
  When valid: For modules that truly export a single primary entity and will never need to export additional items
- Export everything from all modules and rely on documentation to indicate public vs private (rejected)
  Rejected because: This provides no compile-time enforcement of API boundaries and leads to accidental dependencies on internal implementation details
  When valid: Never recommended for production systems with external consumers
- Use a separate public API layer with facade classes that wrap internal implementations (deferred)
  Rejected because: This adds significant complexity and maintenance burden, though it provides maximum flexibility for internal changes
  When valid: For very large systems with strict API stability requirements or when internal and external models diverge significantly

## Risks

- Developers may accidentally export internal utilities, expanding the public API surface unintentionally
  Mitigation: Implement automated linting rules that flag exports from files not in approved public API directories, and require code review for all export changes
  Owner: Engineering team
- Overly strict export discipline may slow down development velocity during rapid iteration phases
  Mitigation: Allow experimental modules with relaxed export rules during development, but require cleanup before merging to main branches
  Owner: Engineering team
- Breaking changes to public APIs may be introduced without proper versioning or migration paths
  Mitigation: Implement API compatibility testing in CI that compares exports against previous versions and flags breaking changes
  Owner: DevOps team

## Implementation Notes

- Use ESLint rules like 'no-restricted-exports' to enforce export patterns and prevent accidental exposure of internal APIs
- Create index.ts files at package boundaries (e.g., src/agents/index.ts) that explicitly re-export all public agent classes
- Document the public API surface in a central API.md file that lists all exported modules and their intended use cases
- Use TypeScript's 'export type' syntax for type-only exports to clearly distinguish runtime exports from compile-time types
- Consider using API Extractor or similar tools to generate API reports and detect breaking changes automatically

## Continuation Context


Verify commands:
- grep -r 'export class.*Agent' src/agents/*.ts | wc -l
- grep -r 'export {' src/agents/index.ts src/core/*.ts
- npx tsc --noEmit && echo 'TypeScript compilation successful'
- find src/agents src/core -name '*.ts' -exec grep -L 'export' {} \; | grep -v '.spec.ts' | grep -v '_internal'

Accept when:
- All agent classes in src/agents/ have explicit named exports and are re-exported from src/agents/index.ts
- Core utility modules (hash.ts, GitignoreUtils.ts, merge.ts) export only their public functions with TypeScript type definitions
- TypeScript compilation succeeds without errors, confirming all export/import relationships are valid
- No internal implementation files (prefixed with _ or in internal/ directories) are exported from public index files

## Enforcement

- Verified by: Automated ESLint rules checking export patterns during pre-commit hooks
- Verified by: TypeScript compiler verification in CI pipeline ensuring type safety of exports
- Verified by: Code review checklist requiring approval for any changes to public API exports
- Verified by: API compatibility tests comparing current exports against baseline snapshots
- Violation handling: CI build fails if ESLint detects unauthorized exports or missing type definitions
- Violation handling: Pull requests modifying public exports are automatically flagged for architecture review
- Violation handling: Breaking changes to public APIs require explicit version bump and migration guide
- Violation handling: Violations discovered in production trigger immediate hotfix process and post-mortem
- Exception process: Developer submits exception request documenting why standard export pattern cannot be followed
- Exception process: Architecture review team evaluates impact on API consumers and long-term maintenance
- Exception process: Approved exceptions must include compensating controls (extra documentation, deprecation timeline)
- Exception process: All exceptions are logged in ADR amendments and reviewed quarterly for potential removal