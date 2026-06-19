# Standardize Subagent Discovery and Contract Propagation via Exported Public APIs: Public Contracts Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all modules implementing subagent discovery, loading, and propagation functionality.

## Context

- The system requires dynamic discovery and loading of subagent configurations from the filesystem, parsing frontmatter metadata from markdown files using js-yaml and @iarna/toml parsers
- Subagent processor and utility modules expose well-defined public contracts (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile, parseFrontmatter, validateFrontmatter, propagateSubagentsForClaude) to coordinate subagent lifecycle operations
- Integration with Claude requires atomic propagation of discovered subagent configurations, necessitating coordination between discovery, validation, and propagation phases
- The architecture separates concerns between SubagentsProcessor (orchestration, discovery, gitignore handling, propagation) and SubagentsUtils (parsing, validation, file loading) to enable independent evolution of discovery logic and parsing logic

## Problem Statement

Without standardized public API contracts for subagent discovery and propagation, modules risk tight coupling, inconsistent validation, and fragile integration points when coordinating filesystem-based subagent configuration loading with external systems like Claude.

## Decision

1. SHOULD: Public contracts SHOULD use TypeScript types (ParsedFrontmatter, CopilotToolMapping) to enforce compile-time validation of subagent metadata structures

## Policy Block

- SHOULD Public contracts SHOULD use TypeScript types (ParsedFrontmatter, CopilotToolMapping) to enforce compile-time validation of subagent metadata structures

In scope:
- All modules in src/core/ that implement subagent discovery, parsing, validation, or propagation
- Integration points with Claude that consume subagent configurations
- Filesystem operations that traverse, read, or write subagent markdown files
- TypeScript type definitions for subagent metadata (ParsedFrontmatter, CopilotToolMapping)

Out of scope:
- Runtime execution of subagent logic (covered by separate execution contracts)
- Network transport protocols for subagent communication
- Authentication and authorization for subagent invocation
- Subagent-specific business logic implementation details

Exceptions:
- EXC-001: Legacy subagent configurations exist that predate TypeScript type enforcement

## Rationale

- The evidence shows explicit separation of concerns between SubagentsProcessor (orchestration) and SubagentsUtils (parsing), with well-defined public contracts enabling independent module evolution and testing
- Use of js-yaml and @iarna/toml for frontmatter parsing indicates standardization on YAML/TOML metadata formats, requiring consistent parsing contracts across all subagent loading operations
- The propagateSubagentsForClaude function and atomic write operations demonstrate integration requirements with external systems, necessitating stable public API contracts to prevent breaking changes
- Detection of message queue boundaries (targets.add, mapped.add) and concurrency model functions indicates coordination requirements that benefit from explicit contract definitions

## Consequences

Positive:
- Clear separation of concerns enables independent testing and evolution of discovery logic (SubagentsProcessor) and parsing logic (SubagentsUtils)
- Standardized public contracts reduce coupling between subagent lifecycle phases and external integrations like Claude
- TypeScript type enforcement (ParsedFrontmatter, CopilotToolMapping) provides compile-time validation of metadata structures, catching errors early
- Atomic write operations and explicit propagation contracts ensure consistent state when coordinating with external systems

Negative:
- Public API contracts create versioning obligations and require careful deprecation management when evolving subagent metadata formats
- Separation across multiple modules (SubagentsProcessor, SubagentsUtils) increases cognitive overhead for developers unfamiliar with the architecture
- Dependency on specific parsing libraries (js-yaml, @iarna/toml) couples the public contracts to these implementations, complicating future parser migrations
- Test-only functions (_resetExperimentalWarningForTests) in public exports may leak implementation details and create maintenance burden

## Alternatives

- Monolithic subagent module with internal functions instead of separated public contracts (rejected)
  Rejected because: Would couple discovery, parsing, and propagation concerns, making independent testing and evolution difficult; evidence shows deliberate separation across SubagentsProcessor and SubagentsUtils
  When valid: For simple systems with single-file subagent configurations and no external integration requirements
- Event-driven architecture with message bus for subagent lifecycle coordination (rejected)
  Rejected because: Evidence shows direct function calls and synchronous coordination patterns; message queue boundaries detected are for internal collection operations (targets.add, mapped.add), not inter-module messaging
  When valid: For distributed systems requiring asynchronous subagent discovery across multiple services or runtime environments
- Plugin-based architecture with dynamic contract discovery via reflection (deferred)
  Rejected because: TypeScript type system and explicit exports indicate preference for compile-time contracts; dynamic discovery would sacrifice type safety
  When valid: If future requirements demand runtime-extensible subagent parsers or third-party subagent format plugins

## Risks

- Breaking changes to public contracts (discoverSubagents, loadSubagentFile, propagateSubagentsForClaude) could cascade failures across dependent modules and external integrations
  Mitigation: Implement semantic versioning for public contracts, maintain backward compatibility for at least one major version, and provide deprecation warnings before removal
  Owner: Core API team
- Tight coupling to js-yaml and @iarna/toml parsers makes migration to alternative parsing libraries expensive and risky
  Mitigation: Introduce parser abstraction layer in SubagentsUtils to isolate parsing implementation from public contracts, enabling future parser swaps without API changes
  Owner: Engineering team
- Atomic write operations (writeAgentsDirectoryAtomic) may fail under high concurrency or filesystem constraints, causing propagation failures to Claude
  Mitigation: Implement retry logic with exponential backoff, add filesystem lock coordination, and provide fallback propagation mechanisms for degraded filesystem scenarios
  Owner: Infrastructure team

## Implementation Notes

- Export all public contracts (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile, parseFrontmatter, validateFrontmatter, propagateSubagentsForClaude) from module index files to establish clear API boundaries
- Use TypeScript's 'export type' for metadata structures (ParsedFrontmatter, CopilotToolMapping) to ensure compile-time validation without runtime overhead
- Implement comprehensive unit tests for each public contract function, mocking filesystem operations and parser dependencies to enable isolated testing
- Document public contracts with JSDoc annotations including parameter types, return types, error conditions, and usage examples to support API consumers
- Consider introducing a SubagentContract interface or type to formalize the expected structure of discovered subagent configurations

## Continuation Context


Verify commands:
- grep -r "export.*discoverSubagents\|export.*loadSubagentFile\|export.*propagateSubagentsForClaude" src/core/
- grep -r "import.*js-yaml\|import.*@iarna/toml" src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts
- npm test -- --testPathPattern="SubagentsProcessor|SubagentsUtils" --coverage

Accept when:
- All public contract functions (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile, parseFrontmatter, validateFrontmatter, propagateSubagentsForClaude) are exported from SubagentsProcessor.ts and SubagentsUtils.ts
- TypeScript compilation succeeds with strict type checking enabled for ParsedFrontmatter and CopilotToolMapping types
- Unit tests achieve >80% coverage for public contract functions with mocked filesystem and parser dependencies

## Enforcement

- Verified by: TypeScript compiler strict mode checks enforce type contracts at build time
- Verified by: Unit test suite validates public contract behavior with mocked dependencies
- Verified by: Code review checklist verifies new subagent operations use standardized public contracts
- Verified by: Integration tests validate propagateSubagentsForClaude contract with Claude system
- Violation handling: TypeScript compilation failures block CI pipeline until type contract violations are resolved
- Violation handling: Code review rejects PRs that bypass public contracts or introduce direct filesystem coupling
- Violation handling: Runtime validation errors in parseFrontmatter/validateFrontmatter are logged with subagent file path and metadata structure for debugging
- Violation handling: Integration test failures trigger rollback of subagent configuration changes
- Exception process: Request architecture review for proposed exceptions to public contract usage
- Exception process: Document exception rationale in ADR amendment with migration timeline
- Exception process: Obtain approval from core API team and affected integration owners (e.g., Claude integration team)
- Exception process: Add exception to policy_exceptions with required approval level and documentation requirements