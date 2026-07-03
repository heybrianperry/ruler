# Standardize Public API Export Patterns for Agent Integration: Agent Modules Registered

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all agent implementations and public API modules. All new agent integrations and external API surfaces MUST comply with these export patterns.

## Context

- The codebase contains multiple agent implementations (GooseAgent, FirebaseAgent, ClineAgent, KiroAgent, AugmentCodeAgent, AntigravityAgent, TraeAgent) that require consistent public API patterns for external integration
- Pattern signature eff7ce064ab2e3f219859e6512c7e811 detected across 12 files with 90% confidence, indicating a deliberate architectural pattern for API design
- Core utilities (GitignoreUtils, hash.ts) and integration modules (mcp/merge.ts) demonstrate consistent export patterns that enable external consumption
- The agent index file serves as a central registry, requiring standardized exports from individual agent modules to maintain a cohesive public API surface
- External consumers and test harnesses depend on predictable, well-structured API contracts for agent discovery and invocation

## Problem Statement

Without standardized public API export patterns, agent implementations risk inconsistent interfaces, making integration difficult for external consumers. The system needs a unified approach to exposing agent capabilities, utility functions, and integration points that ensures discoverability, type safety, and maintainability across the growing agent ecosystem.

## Decision

1. MUST: Agent modules MUST be registered in the central index file (src/agents/index.ts) to enable discovery

## Policy Block

- MUST Agent modules MUST be registered in the central index file (src/agents/index.ts) to enable discovery

In scope:
- All agent implementation files in src/agents/
- Core utility modules in src/core/ that provide public functionality
- Integration modules in src/mcp/ and similar directories
- Index files that aggregate and re-export agent implementations
- Test harness modules that consume agent APIs

Out of scope:
- Internal implementation details not exposed through public exports
- Private helper functions within agent modules
- Configuration files and build scripts
- Development-only utilities and debugging tools

Exceptions:
- EXC-001: Legacy agent implementations may temporarily use default exports during migration period
- EXC-002: Experimental or prototype agents in development may defer index registration

## Rationale

- Pattern detected across 12 files with 90% confidence indicates this is an established architectural convention that has proven effective
- Consistent export patterns enable automated tooling, IDE support, and reduce cognitive load when working across multiple agent implementations
- Central registration through index files provides a single source of truth for agent discovery, simplifying external integration and testing
- Explicit named exports improve refactoring safety and make dependencies explicit, reducing the risk of breaking changes in public APIs

## Consequences

Positive:
- Improved discoverability of agent capabilities through centralized registration and consistent naming
- Enhanced type safety and IDE autocomplete support for external consumers
- Simplified testing through predictable API contracts and clear dependency boundaries
- Reduced integration friction for new agent implementations following established patterns
- Better maintainability through explicit exports and standardized module structure

Negative:
- Additional boilerplate required for each new agent implementation to maintain consistency
- Central index file becomes a potential bottleneck requiring updates for each new agent
- Migration effort required for any existing agents using non-standard export patterns
- Stricter conventions may feel restrictive for rapid prototyping or experimental features

## Alternatives

- Use default exports for all agent modules to reduce verbosity (rejected)
  Rejected because: Default exports reduce refactoring safety, complicate re-exports in index files, and provide poor IDE support for auto-imports
  When valid: May be reconsidered if TypeScript tooling significantly improves default export handling
- Allow each agent to define its own API contract without standardization (rejected)
  Rejected because: Lack of standardization increases integration complexity, makes testing harder, and creates inconsistent developer experience
  When valid: Could be valid for truly unique agents with fundamentally different interaction models
- Use a plugin registry system with runtime registration instead of static index (deferred)
  Rejected because: Adds runtime complexity and reduces type safety, though may be needed for dynamic plugin loading in future
  When valid: Should be reconsidered if the system needs to support third-party agent plugins loaded at runtime

## Risks

- Central index file becomes a merge conflict hotspot as team scales
  Mitigation: Implement automated index generation tooling or adopt alphabetical ordering convention to minimize conflicts
  Owner: Engineering team
- Overly rigid export patterns may hinder innovation or special-case agent implementations
  Mitigation: Provide clear exception process and regularly review patterns for necessary evolution based on real-world usage
  Owner: Architecture team
- Breaking changes to common agent interface affect all implementations simultaneously
  Mitigation: Use versioned interfaces and deprecation periods for interface changes, maintain backward compatibility where possible
  Owner: API design team

## Implementation Notes

- Create a base Agent interface or abstract class that defines the common contract all agents must implement
- Use TypeScript's 'export type' and 'export interface' to clearly separate type exports from value exports
- Consider using barrel exports (index.ts files) at the agent directory level to simplify imports for consumers
- Document the agent registration process in CONTRIBUTING.md with step-by-step instructions for new agent authors
- Implement ESLint rules to enforce named exports and detect unregistered agent modules

## Continuation Context


Verify commands:
- grep -r 'export default' src/agents/*.ts | grep -v '.test.ts' | wc -l | grep -q '^0$'
- grep -r 'export class.*Agent' src/agents/*.ts | wc -l
- test -f src/agents/index.ts && grep -q 'export.*Agent' src/agents/index.ts

Accept when:
- No default exports found in agent implementation files (excluding test files)
- All agent classes follow the naming convention with 'Agent' suffix
- Central index file exists and contains exports for all agent implementations

## Enforcement

- Verified by: ESLint rules checking for named exports in agent modules
- Verified by: CI pipeline verification that all agents are registered in index file
- Verified by: TypeScript compiler checks ensuring interface compliance
- Verified by: Code review checklist item for new agent implementations
- Violation handling: CI build fails if agent modules use default exports
- Violation handling: PR review blocks merge if new agents are not registered in index
- Violation handling: Automated comments on PRs identifying missing type definitions
- Violation handling: Quarterly audit reports identifying non-compliant modules for remediation
- Exception process: Submit exception request via architecture review board with justification
- Exception process: Document exception in agent module header comments with ADR reference
- Exception process: Include migration plan and timeline if exception is temporary
- Exception process: Review exceptions quarterly to determine if they can be resolved or if patterns need updating