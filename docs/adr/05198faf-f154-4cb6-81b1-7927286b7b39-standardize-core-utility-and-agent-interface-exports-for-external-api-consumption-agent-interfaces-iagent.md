# Standardize Core Utility and Agent Interface Exports for External API Consumption: Agent Interfaces Iagent

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all public API modules, core utilities, and agent interfaces exposed to external consumers. It applies to all TypeScript modules in src/core/, src/agents/, and any module designated as part of the public API surface.

## Context

- The codebase contains multiple core utility modules (SkillsUtils, GitignoreUtils, RuleProcessor, SkillsProcessor) and agent interfaces (IAgent, JulesAgent, KiroAgent, OpenHandsAgent) that form the public API surface
- External consumers and integrations require stable, well-defined interfaces to interact with the system's core functionality and agent capabilities
- Pattern signature eff7ce064ab2e3f219859e6512c7e811 detected across 10 files with 90% confidence indicates a consistent architectural approach to API design
- The UnifiedConfigTypes module suggests a centralized configuration strategy that needs to be exposed through public APIs
- Agent selection logic (agent-selection.ts) indicates a plugin-like architecture where external systems may need to interact with or extend agent capabilities

## Problem Statement

Without standardized patterns for exposing core utilities and agent interfaces as public APIs, external consumers face inconsistent integration experiences, unclear API boundaries, and potential breaking changes. The system needs a coherent strategy for defining which modules constitute the public API surface, how they should be structured, and what stability guarantees they provide to external integrators.

## Decision

1. MUST: Agent interfaces (IAgent and implementations) MUST define stable method signatures that external plugins can reliably implement or extend

## Policy Block

- MUST Agent interfaces (IAgent and implementations) MUST define stable method signatures that external plugins can reliably implement or extend

In scope:
- All TypeScript modules in src/core/ directory
- All agent interface definitions and implementations in src/agents/
- Configuration type definitions in UnifiedConfigTypes
- Agent selection and registration mechanisms
- Utility functions explicitly marked for external consumption

Out of scope:
- Internal helper functions not exported from module index files
- Test utilities and mock implementations
- Build-time code generation scripts
- Development-only debugging utilities
- Private implementation classes not exposed through public interfaces

Exceptions:
- EXC-001: A module is explicitly marked as experimental or alpha-quality with clear documentation warnings
- EXC-002: Breaking changes are required for critical security fixes

## Rationale

- Pattern detected across 10 files with 90% confidence indicates this is an established architectural approach that should be formalized
- Consistent API design patterns reduce integration friction and improve developer experience for external consumers
- Explicit interface definitions enable better tooling support, type safety, and documentation generation
- Separating public API surface from internal implementation provides flexibility to refactor internals without breaking external integrations

## Consequences

Positive:
- External integrators gain stable, well-documented APIs with clear contracts and versioning guarantees
- TypeScript type definitions provide compile-time safety and excellent IDE autocomplete support
- Clear API boundaries enable internal refactoring without breaking external consumers
- Plugin architecture for agents allows ecosystem growth without core modifications

Negative:
- Maintaining API stability requires additional discipline and may slow down internal refactoring
- Deprecation cycles add overhead to the development process
- Public API surface requires comprehensive documentation and examples
- Version management complexity increases with multiple public API modules

## Alternatives

- Expose all internal modules as public API without explicit interface definitions (rejected)
  Rejected because: Would create tight coupling between external consumers and internal implementation details, making refactoring nearly impossible and leading to widespread breakage
  When valid: Never appropriate for production systems with external integrators
- Use a single monolithic API facade that wraps all functionality (rejected)
  Rejected because: Would create a bottleneck for API evolution and make it difficult to version different subsystems independently. Also reduces tree-shaking opportunities for consumers
  When valid: May be appropriate for very small libraries with minimal functionality
- Adopt a plugin-only architecture where all functionality is provided through plugins (deferred)
  Rejected because: Would require significant architectural changes and may be overly complex for current needs
  When valid: Could be reconsidered if the number of external integrations grows significantly and requires more flexibility

## Risks

- API surface grows too large and becomes difficult to maintain with stability guarantees
  Mitigation: Regularly review public API surface and deprecate unused functionality. Use feature flags for experimental APIs
  Owner: API Architecture Team
- External consumers depend on undocumented behavior or implementation details
  Mitigation: Comprehensive API documentation, clear examples, and automated API compatibility testing in CI pipeline
  Owner: Engineering Team
- Breaking changes slip through without proper deprecation cycles
  Mitigation: Implement automated API surface comparison tools and require explicit approval for any public API changes
  Owner: Release Engineering Team

## Implementation Notes

- Create index.ts files in src/core/ and src/agents/ that explicitly export public API members
- Add JSDoc comments with @public, @beta, or @experimental tags to all exported interfaces and functions
- Implement API extractor or similar tooling to generate API documentation and detect breaking changes
- Establish a public API review process where changes to exported interfaces require architecture team approval
- Create integration tests that verify public API contracts from an external consumer perspective

## Continuation Context


Verify commands:
- grep -r "export.*interface" src/core/ src/agents/ | wc -l
- npx api-extractor run --local || echo 'API surface validation'
- grep -r "@public\|@beta\|@experimental" src/core/ src/agents/ | wc -l

Accept when:
- All modules in src/core/ and src/agents/ have explicit index.ts files exporting public interfaces
- JSDoc tags (@public, @beta, @experimental) are present on at least 80% of exported members
- API extractor or equivalent tooling runs successfully in CI and detects no undocumented breaking changes
- Integration test suite exists that validates public API contracts from external consumer perspective

## Enforcement

- Verified by: Automated API surface comparison in CI pipeline using api-extractor or similar tooling
- Verified by: Code review checklist requiring architecture approval for changes to public API modules
- Verified by: TypeScript compiler strict mode ensuring type safety of exported interfaces
- Verified by: Integration test suite running against published package versions
- Violation handling: CI pipeline fails if undocumented breaking changes are detected in public API surface
- Violation handling: Pull requests modifying public APIs without proper JSDoc documentation are automatically flagged for review
- Violation handling: Deprecation warnings are logged at runtime when deprecated APIs are used
- Violation handling: Major version bump is required for any breaking changes to public API contracts
- Exception process: Submit exception request to architecture review board with justification and impact analysis
- Exception process: For experimental APIs, clearly document instability in README and package.json keywords
- Exception process: Security-critical breaking changes follow expedited review process with mandatory migration guide
- Exception process: All exceptions are documented in CHANGELOG.md with migration instructions