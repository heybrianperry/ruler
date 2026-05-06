# Standardize Public API Export Patterns for External Integration: Core Library Modules

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all public API modules and external integration points within the codebase.

## Context

- The codebase contains 20 files implementing public API patterns with 90% consistency, indicating a well-established architectural convention
- Multiple agent implementations (CursorAgent, AmpAgent, AgentsMdAgent, FactoryDroidAgent, FirebaseAgent, AiderAgent, KiloCodeAgent) follow a common public API export pattern
- Core library modules (lib.ts, RuleProcessor.ts, SkillsProcessor.ts, SkillsUtils.ts) expose functionality through standardized external interfaces
- CLI commands, MCP merge utilities, and test harnesses require consistent public API contracts for integration
- The pattern signature eff7ce064ab2e3f219859e6512c7e811 represents a stable architectural decision with high significance (90%) across the codebase

## Problem Statement

Without standardized public API export patterns, external integrations become fragile, inconsistent, and difficult to maintain. Different modules exposing functionality through ad-hoc interfaces create integration complexity, increase cognitive load for consumers, and make it difficult to enforce versioning, backward compatibility, and contract stability across the system.

## Decision

1. MUST: Core library modules (lib.ts, RuleProcessor.ts, SkillsProcessor.ts, SkillsUtils.ts) MUST maintain stable public API surfaces that external consumers can depend on

## Policy Block

- MUST Core library modules (lib.ts, RuleProcessor.ts, SkillsProcessor.ts, SkillsUtils.ts) MUST maintain stable public API surfaces that external consumers can depend on

In scope:
- All agent implementations (CursorAgent, AmpAgent, AgentsMdAgent, FactoryDroidAgent, FirebaseAgent, AiderAgent, KiloCodeAgent)
- Core library modules (lib.ts, RuleProcessor.ts, SkillsProcessor.ts, SkillsUtils.ts)
- CLI command interfaces (cli/commands.ts)
- MCP integration utilities (mcp/merge.ts)
- Test harness interfaces (tests/harness.ts)
- Any module intended for external consumption or cross-module integration

Out of scope:
- Internal utility functions not intended for external use
- Private implementation details within module boundaries
- Temporary or experimental APIs marked as internal
- Test-only utilities not part of the production API surface

Exceptions:
- EXC-001: A module is explicitly marked as internal-only and documented as not part of the public API contract
- EXC-002: Rapid prototyping or experimental features require temporary API instability

## Rationale

- Pattern detected across 20 files with 90% confidence indicates this is an established, successful architectural convention worth codifying
- Consistent public API patterns reduce integration complexity and cognitive load for developers working across multiple modules
- Standardized exports enable better tooling support for API documentation, type checking, and automated contract testing
- Clear public/private boundaries improve maintainability by allowing internal refactoring without breaking external consumers

## Consequences

Positive:
- External integrations become more stable and predictable with well-defined API contracts
- Developers can confidently consume public APIs knowing they follow consistent patterns across the codebase
- Automated tooling can generate documentation, type definitions, and contract tests from standardized exports
- Internal refactoring becomes safer as public API boundaries are clearly defined and protected

Negative:
- Additional discipline required to maintain public/private boundaries and avoid leaking implementation details
- Versioning and deprecation processes add overhead when evolving public APIs
- May slow down rapid prototyping if developers must consider API stability too early
- Requires ongoing vigilance to prevent API surface bloat as new features are added

## Alternatives

- Allow ad-hoc, inconsistent export patterns across modules with no standardization (rejected)
  Rejected because: Creates integration fragility, increases cognitive load, and makes it difficult to maintain backward compatibility across the system
  When valid: Never valid for production code; only acceptable in throwaway prototypes
- Use a facade pattern with a single centralized API gateway for all external integrations (rejected)
  Rejected because: Creates a bottleneck, reduces modularity, and makes it difficult to evolve individual components independently
  When valid: May be appropriate for specific high-level integration points but not as a replacement for module-level API standards
- Generate public APIs automatically from implementation code using tooling (deferred)
  Rejected because: Not rejected, but deferred pending evaluation of code generation tooling maturity
  When valid: Could complement this ADR if tooling can enforce the standardized patterns automatically

## Risks

- Existing code may not fully comply with standardized patterns, requiring significant refactoring effort
  Mitigation: Implement gradual migration strategy with automated detection of non-compliant modules; prioritize high-traffic APIs first
  Owner: Engineering team with architecture review oversight
- Overly rigid API standards may stifle innovation or make it difficult to experiment with new patterns
  Mitigation: Provide exception process for experimental features; review and evolve standards quarterly based on team feedback
  Owner: Architecture review team
- Public API versioning and deprecation processes may be ignored under deadline pressure
  Mitigation: Implement automated checks in CI to detect breaking changes; require explicit approval for API modifications
  Owner: DevOps and engineering team

## Implementation Notes

- Start by documenting the existing pattern detected across the 20 compliant files as the reference implementation
- Create linting rules or static analysis checks to detect deviations from the standardized export pattern
- Provide code templates and examples for common public API scenarios (agent implementations, core utilities, CLI commands)
- Establish a review checklist for pull requests that modify public API surfaces to ensure compliance with this ADR

## Continuation Context


Verify commands:
- grep -r 'export.*class.*Agent' src/agents/ | wc -l
- grep -r 'export {' src/lib.ts src/core/*.ts | grep -v 'export { default }'
- npm run lint -- --rule 'no-restricted-syntax: [error, ExportAllDeclaration]'

Accept when:
- All agent implementations export their public interfaces using the standardized pattern
- Core library modules (lib.ts, RuleProcessor.ts, SkillsProcessor.ts, SkillsUtils.ts) have explicit export statements for public APIs
- Linting rules pass with no violations of public API export standards

## Enforcement

- Verified by: Automated linting rules in CI pipeline checking for consistent export patterns
- Verified by: Code review checklist requiring verification of public API compliance
- Verified by: Static analysis tools detecting public/private boundary violations
- Violation handling: CI build fails if linting rules detect non-compliant export patterns
- Violation handling: Pull requests modifying public APIs require architecture review approval
- Violation handling: Quarterly audits identify and prioritize remediation of non-compliant modules
- Exception process: Submit exception request to architecture review team with justification
- Exception process: Document exception in module header with @internal or @experimental tag
- Exception process: Set sunset date for experimental exceptions; review quarterly for internal-only exceptions