# Standardize Agent Public API Contracts with Consistent Interface Patterns: Agent Follow Semantic

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all agent implementations and public API contracts within the system. All new agents and API integrations MUST comply with these interface standards.

## Context

- The system contains multiple agent implementations (JulesAgent, JetBrainsAiAssistantAgent, FirebaseAgent, AntigravityAgent, KiloCodeAgent, AiderAgent, JunieAgent) that expose public APIs for external integration
- Pattern signature dc1b57bd689dc0475c8072de90be279a was detected across 8 files with 90% confidence, indicating a consistent architectural approach to public API contracts
- Agent-based architectures require standardized interfaces to enable interoperability, composability, and consistent integration patterns across different agent types
- The SkillsUtils core module suggests a centralized approach to managing agent capabilities and skills through standardized contracts
- External consumers and internal orchestration layers depend on predictable, well-defined API contracts for reliable agent interaction

## Problem Statement

Without standardized public API contracts for agent implementations, the system faces integration fragmentation, inconsistent error handling, unpredictable interface evolution, and increased maintenance burden. Each agent may expose different contract patterns, making it difficult for consumers to interact uniformly with multiple agents and reducing the overall composability of the agent ecosystem.

## Decision

1. SHOULD: Agent APIs SHOULD follow semantic versioning principles and maintain backward compatibility for public contract changes

## Policy Block

- SHOULD Agent APIs SHOULD follow semantic versioning principles and maintain backward compatibility for public contract changes

In scope:
- All agent implementations in the /src/agents directory
- Core utility modules that define agent interfaces (e.g., SkillsUtils)
- Public API endpoints exposed to external consumers or orchestration layers
- Agent registration and discovery mechanisms
- Cross-agent communication and integration patterns

Out of scope:
- Internal agent implementation details not exposed through public APIs
- Private helper methods and utilities used only within agent boundaries
- Third-party library APIs consumed by agents but not re-exposed
- Development and testing utilities not intended for production use

Exceptions:
- EXC-001: Legacy agents implemented before this ADR was established may temporarily deviate from the standard contract
- EXC-002: Experimental or prototype agents in early development phases

## Rationale

- The detection of this pattern across 8 agent files with 90% confidence indicates an established architectural practice that should be formalized and enforced
- Standardized public API contracts reduce integration complexity, enable agent composability, and improve maintainability by providing predictable interfaces
- Centralized skill registration through SkillsUtils enables dynamic agent discovery and orchestration capabilities essential for scalable agent ecosystems
- Consistent error handling and type safety across agent APIs improves developer experience and reduces runtime failures in production environments

## Consequences

Positive:
- Improved interoperability between different agent implementations through consistent interface patterns
- Reduced integration effort for consumers who can rely on predictable API contracts across all agents
- Enhanced maintainability through centralized interface definitions and standardized patterns
- Better developer experience with clear type definitions, documentation, and consistent error handling
- Easier testing and mocking of agent interactions due to well-defined contracts

Negative:
- Initial refactoring effort required to bring existing agents into compliance with standardized contracts
- Potential constraints on agent-specific optimizations or unique interface requirements
- Additional overhead in maintaining interface documentation and versioning
- Learning curve for developers unfamiliar with the standardized contract patterns

## Alternatives

- Allow each agent to define its own unique public API contract without standardization (rejected)
  Rejected because: This approach leads to integration fragmentation, increased maintenance burden, and poor composability across the agent ecosystem. The detection of a consistent pattern across 8 files indicates that standardization is already emerging organically and should be formalized.
  When valid: Only appropriate for isolated, single-purpose agents with no integration requirements
- Implement a strict base class or abstract interface that all agents must inherit from (deferred)
  Rejected because: While this provides strong enforcement, it may be too rigid for the current TypeScript-based architecture. The current pattern-based approach offers flexibility while maintaining consistency.
  When valid: Could be reconsidered if the agent ecosystem grows significantly and requires stronger compile-time guarantees
- Use a protocol-based approach with runtime validation instead of compile-time contracts (rejected)
  Rejected because: Runtime validation alone provides weaker guarantees and catches errors later in the development cycle. TypeScript's type system enables compile-time contract verification which is more robust.
  When valid: May be appropriate for dynamic plugin systems where agents are loaded at runtime from external sources

## Risks

- Existing agents may have significant technical debt or architectural differences that make contract standardization difficult or expensive
  Mitigation: Implement a phased migration approach with clear milestones. Provide exception process for legacy agents with documented migration plans. Create migration tooling and templates to reduce refactoring effort.
  Owner: Engineering team with architecture review board oversight
- Overly rigid contract standards may stifle innovation or prevent agent-specific optimizations
  Mitigation: Design contracts with extension points (R-25-008) that allow agent-specific enhancements. Establish a governance process for contract evolution based on real-world usage patterns.
  Owner: Architecture review board
- Contract versioning and backward compatibility may become complex as the agent ecosystem evolves
  Mitigation: Adopt semantic versioning from the start (R-25-004). Implement automated contract compatibility testing. Maintain clear deprecation policies and migration guides for breaking changes.
  Owner: Platform engineering team

## Implementation Notes

- Start by documenting the existing contract pattern detected across the 8 agent files as the baseline standard
- Create a reference implementation or template agent that demonstrates full compliance with all contract requirements
- Develop TypeScript interfaces or type definitions that codify the standard agent contract and can be imported by all agent implementations
- Implement automated linting or static analysis rules that verify agent compliance with contract standards during CI/CD
- Establish a migration priority based on agent usage and external exposure, focusing first on agents with the most external consumers

## Continuation Context


Verify commands:
- grep -r "export.*Agent" src/agents/ | wc -l  # Verify all agents export consistent interface
- find src/agents -name "*.ts" -exec grep -L "SkillsUtils" {} \; | wc -l  # Check for agents not using SkillsUtils
- npm run type-check  # Verify TypeScript compilation with strict type checking

Accept when:
- All agent files in src/agents/ export a consistent public API contract with documented types
- All agents register their capabilities through SkillsUtils or equivalent centralized mechanism
- TypeScript compilation succeeds with strict mode enabled and no type errors in agent public APIs
- API documentation (OpenAPI/Swagger or equivalent) exists for all public agent endpoints

## Enforcement

- Verified by: Automated TypeScript type checking in CI/CD pipeline with strict mode enabled
- Verified by: Static analysis tools (ESLint with custom rules) that verify agent contract compliance
- Verified by: Code review checklist items specifically covering public API contract requirements
- Verified by: Integration tests that validate agent interface consistency across all implementations
- Violation handling: CI/CD pipeline fails if TypeScript compilation errors occur due to contract violations
- Violation handling: Pull requests are blocked if static analysis detects non-compliant agent implementations
- Violation handling: Code review process requires explicit acknowledgment of contract compliance or documented exception
- Violation handling: Quarterly architecture reviews audit agent compliance and track migration progress for legacy agents
- Exception process: Submit exception request to architecture review board with justification and impact analysis
- Exception process: Provide documented migration plan with timeline for achieving compliance
- Exception process: Mark non-compliant agents as experimental or internal-only in documentation
- Exception process: Review exceptions quarterly to ensure migration plans are progressing and exceptions remain valid