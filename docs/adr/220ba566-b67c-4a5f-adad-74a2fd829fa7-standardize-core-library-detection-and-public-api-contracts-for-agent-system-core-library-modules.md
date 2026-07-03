# Standardize Core Library Detection and Public API Contracts for Agent System: Core Library Modules

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent implementations and core library modules within the system.

## Context

- The system implements multiple agent types (GeminiCliAgent, JulesAgent, JunieAgent, KiroAgent, GooseAgent, KiloCodeAgent, JetBrainsAiAssistantAgent) that share common interfaces and configuration patterns through AbstractAgent and IAgent contracts
- Core library modules (UnifiedConfigLoader, GitignoreUtils, SkillsProcessor, RuleProcessor, FileSystemUtils) provide foundational services using Node.js fs, path, and fs/promises APIs with consistent import patterns
- Public API contracts are explicitly exported from modules to establish clear boundaries between agent implementations and core utilities, enabling modular composition and independent evolution
- Configuration loading involves multiple formats (TOML via @iarna/toml, JSON) with validation through JSON.parse and parseTOML operations, requiring standardized input validation patterns
- The system supports concurrent operations through async functions (loadUnifiedConfig, discoverSkills, propagateSkills, applyRulerConfig) that coordinate file system access and configuration processing

## Problem Statement

Without standardized patterns for core library detection and public API contract definition, agent implementations and core utilities risk inconsistent dependency management, unclear module boundaries, and fragile integration points that complicate testing, maintenance, and extension of the agent system.

## Decision

1. MUST: Core library modules MUST explicitly declare public API contracts through exported types and functions (e.g., IAgent, IAgentConfig, loadUnifiedConfig, updateGitignore)

## Policy Block

- MUST Core library modules MUST explicitly declare public API contracts through exported types and functions (e.g., IAgent, IAgentConfig, loadUnifiedConfig, updateGitignore)

In scope:
- All agent implementation files in src/agents/ directory
- All core utility modules in src/core/ directory
- CLI handlers in src/cli/ that coordinate agent operations
- Configuration loading and processing modules
- File system utility functions and gitignore management

Out of scope:
- Third-party library internal implementations
- Test fixtures and mock implementations
- Build and deployment scripts
- Documentation generation tools

Exceptions:
- EXC-001: Legacy agent implementations require migration period to adopt IAgent interface

## Rationale

- The evidence shows 25 files with 90.24% confidence exhibiting consistent patterns of core library imports (fs, path, fs/promises) and public contract exports, indicating an established architectural convention
- Explicit public API contracts (IAgent, IAgentConfig, loadUnifiedConfig, updateGitignore) enable clear separation between agent implementations and core utilities, supporting independent testing and evolution
- Standardized input validation through JSON.parse and parseTOML operations across configuration loaders (UnifiedConfigLoader, GeminiCliAgent) reduces security risks and parsing inconsistencies
- The concurrency model demonstrated by async functions (loadUnifiedConfig, discoverSkills, propagateSkills) provides a proven pattern for coordinating file system operations without race conditions

## Consequences

Positive:
- Clear module boundaries through explicit public API contracts improve testability and enable independent evolution of agent implementations
- Consistent core library usage (fs, path, fs/promises) reduces cognitive load and simplifies onboarding for new contributors
- Standardized input validation patterns reduce security vulnerabilities from malformed configuration files
- Established concurrency patterns prevent race conditions in file system operations across multiple agents

Negative:
- Strict adherence to IAgent and AbstractAgent contracts may require refactoring existing agent implementations that deviate from the pattern
- Dependency on Node.js fs/path APIs limits portability to non-Node.js runtimes without abstraction layers
- Mandatory input validation through JSON.parse adds parsing overhead for trusted internal configuration sources
- Concurrency model assumptions may not fit all future agent coordination patterns, requiring pattern evolution

## Alternatives

- Allow each agent implementation to define its own interfaces and core library usage patterns without standardization (rejected)
  Rejected because: Evidence shows 25 files already converging on consistent patterns; lack of standardization would increase maintenance burden and integration complexity
  When valid: Only valid for experimental or prototype agents explicitly marked as non-production
- Abstract file system operations behind a custom wrapper instead of using Node.js fs/path directly (deferred)
  Rejected because: Current evidence shows direct fs/path usage is working effectively; abstraction layer would add complexity without clear immediate benefit
  When valid: Should be reconsidered if multi-runtime support (Deno, Bun) becomes a requirement or if file system mocking needs improvement
- Use dependency injection framework to manage core library dependencies instead of direct imports (rejected)
  Rejected because: Direct import pattern is simpler, more transparent, and sufficient for current system complexity; DI framework would add significant overhead
  When valid: Valid only if system grows to require complex runtime configuration of core library implementations

## Risks

- Breaking changes to IAgent or AbstractAgent interfaces could require coordinated updates across all 25+ agent implementations
  Mitigation: Implement interface versioning strategy and maintain backward compatibility shims during transition periods; use TypeScript strict mode to catch breaking changes at compile time
  Owner: Engineering team / Architecture review board
- Direct dependency on Node.js fs/path APIs creates tight coupling to Node.js runtime, limiting future platform flexibility
  Mitigation: Document runtime assumptions clearly; monitor for multi-runtime requirements; prepare abstraction layer design if portability becomes necessary
  Owner: Engineering team
- Inconsistent error handling across JSON.parse and parseTOML validation points could lead to unclear failure modes
  Mitigation: Standardize error handling patterns in configuration loaders; implement comprehensive error logging with context; add validation error tests
  Owner: Core utilities team

## Implementation Notes

- New agent implementations should extend AbstractAgent or AgentsMdAgent and implement the IAgent interface, importing from './AbstractAgent' and './IAgent' respectively
- Core utility modules should explicitly export public contracts as named exports (e.g., export { loadUnifiedConfig, UnifiedLoadOptions }) to establish clear API boundaries
- Configuration parsing functions should wrap JSON.parse and parseTOML calls in try-catch blocks with descriptive error messages including file paths and parse positions
- Async file system operations should follow the pattern established by loadUnifiedConfig and discoverSkills: use fs/promises, handle errors explicitly, and document concurrency assumptions
- When adding new core libraries, update the libs.core.detected pattern documentation and ensure consistent import patterns across related modules

## Continuation Context


Verify commands:
- grep -r "import.*from.*['\"]\./IAgent['\"]" src/agents/ | wc -l
- grep -r "export.*{.*}" src/core/*.ts | grep -E "(loadUnifiedConfig|updateGitignore|discoverSkills|concatenateRules)"
- grep -r "JSON\.parse\|parseTOML" src/core/*.ts src/agents/*.ts
- find src/agents -name "*.ts" -exec grep -l "extends.*Agent" {} \;

Accept when:
- All agent implementation files in src/agents/ import IAgent or AbstractAgent interfaces
- Core utility modules in src/core/ explicitly export public API contracts as named exports
- Configuration parsing operations use JSON.parse or parseTOML with error handling
- File system operations consistently use Node.js fs, path, or fs/promises modules

## Enforcement

- Verified by: TypeScript compiler checks for interface compliance and import resolution
- Verified by: Code review checklist verifying IAgent interface implementation for new agents
- Verified by: Static analysis tools scanning for direct fs/path imports and public export patterns
- Verified by: CI pipeline grep-based verification commands checking pattern compliance
- Violation handling: TypeScript compilation failures block merge for missing interface implementations
- Violation handling: Code review feedback requests refactoring to align with established patterns
- Violation handling: CI pipeline warnings flag deviations from core library usage patterns
- Violation handling: Architecture review required for intentional deviations with documented justification
- Exception process: Submit exception request documenting specific technical constraints preventing pattern compliance
- Exception process: Architecture review board evaluates exception against system-wide consistency goals
- Exception process: Approved exceptions documented in code comments with ADR reference and expiration timeline
- Exception process: Quarterly review of active exceptions to assess migration opportunities