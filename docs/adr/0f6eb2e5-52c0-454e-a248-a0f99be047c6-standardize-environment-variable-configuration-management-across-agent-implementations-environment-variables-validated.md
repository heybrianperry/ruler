# Standardize Environment Variable Configuration Management Across Agent Implementations: Environment Variables Validated

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all agent implementations and configuration management components within the system. All new agents and configuration loaders MUST comply with these rules from their initial implementation.

## Context

- The codebase contains multiple agent implementations (ZedAgent, GeminiCliAgent, RooCodeAgent, OpenCodeAgent, QwenCodeAgent) that require runtime configuration through environment variables
- A UnifiedConfigLoader component exists to centralize configuration loading logic, indicating a need for consistent configuration management patterns
- The apply-engine and MCP propagation components also interact with environment configuration, suggesting cross-cutting configuration concerns
- Pattern detected across 8 files with 91.20% confidence indicates a well-established architectural approach to environment and configuration management
- The paradigm.concurrency_model facet suggests configuration management impacts how concurrent operations access and share configuration state

## Problem Statement

Without standardized environment variable configuration management, agent implementations may handle configuration inconsistently, leading to runtime errors, difficult debugging, security vulnerabilities from improper secret handling, and maintenance overhead when configuration requirements change across multiple agent types.

## Decision

1. MUST: Environment variables MUST be validated at application startup before agent initialization to fail fast on misconfiguration

## Policy Block

- MUST Environment variables MUST be validated at application startup before agent initialization to fail fast on misconfiguration

In scope:
- All agent implementations (ZedAgent, GeminiCliAgent, RooCodeAgent, OpenCodeAgent, QwenCodeAgent, and future agents)
- Core configuration components (UnifiedConfigLoader, apply-engine)
- MCP propagation and integration components that require configuration
- Test fixtures and test environments that simulate production configuration

Out of scope:
- Third-party library configuration that does not integrate with the agent system
- Build-time configuration and compilation flags
- Static configuration files (JSON, YAML) that are bundled with the application
- User-provided runtime parameters passed through CLI arguments or API requests

Exceptions:
- EXC-001: Legacy agent implementations being migrated to the new configuration system
- EXC-002: Emergency hotfix scenarios where configuration validation would block critical production fixes

## Rationale

- Centralized configuration management reduces code duplication across 8+ agent implementations and ensures consistent behavior
- Immutable configuration state prevents concurrency bugs in multi-threaded or async execution contexts, which is critical given the paradigm.concurrency_model facet
- Early validation and fail-fast behavior improves developer experience by catching configuration errors before runtime execution
- Abstraction layer over direct environment variable access enables future migration to alternative configuration sources (secret managers, config services) without changing agent code

## Consequences

Positive:
- Reduced maintenance burden when adding new configuration parameters or changing existing ones
- Improved security posture through centralized secret handling and validation
- Better testability through dependency injection of configuration rather than global environment state
- Consistent error messages and debugging experience across all agent implementations

Negative:
- Additional abstraction layer adds slight complexity and indirection to configuration access
- Migration effort required for any existing agents that directly access environment variables
- Potential performance overhead from validation and immutability enforcement, though typically negligible
- Learning curve for developers unfamiliar with the UnifiedConfigLoader pattern

## Alternatives

- Allow each agent to directly access process.env with no centralized management (rejected)
  Rejected because: Leads to inconsistent validation, duplicated code, difficult testing, and no protection against concurrent modification issues
  When valid: Only appropriate for trivial single-agent prototypes or proof-of-concept code
- Use a third-party configuration library (e.g., dotenv, config, convict) without custom wrapper (rejected)
  Rejected because: Generic libraries lack domain-specific validation for agent requirements and do not enforce immutability guarantees needed for concurrent execution
  When valid: Could be reconsidered if a library emerges that specifically addresses agent concurrency patterns
- Implement configuration as a singleton service with lazy loading (rejected)
  Rejected because: Lazy loading delays error detection and singleton pattern complicates testing; eager validation at startup is preferred
  When valid: May be appropriate for optional configuration that is rarely used and expensive to validate

## Risks

- Configuration loading failures at startup could prevent application from starting in production environments
  Mitigation: Implement graceful degradation for non-critical configuration and comprehensive startup health checks with clear error messages
  Owner: Platform Engineering Team
- Immutable configuration prevents runtime reconfiguration for operational scenarios like credential rotation
  Mitigation: Design configuration reload mechanism with proper synchronization or implement graceful restart procedures for configuration updates
  Owner: Engineering Team
- Centralized configuration component becomes a bottleneck or single point of failure
  Mitigation: Keep configuration loader stateless and lightweight; ensure it has comprehensive test coverage and monitoring
  Owner: Engineering Team

## Implementation Notes

- Start by auditing all agent implementations to identify current environment variable usage patterns and create a comprehensive configuration schema
- Implement UnifiedConfigLoader with TypeScript interfaces that provide type safety and IDE autocomplete for configuration access
- Add integration tests that verify configuration validation catches common misconfiguration scenarios (missing required vars, invalid formats, etc.)
- Document all required and optional environment variables in a central README or configuration guide with examples for each environment type
- Consider implementing a configuration validation CLI tool that can be run in CI/CD pipelines to catch configuration drift

## Continuation Context


Verify commands:
- grep -r 'process\.env' src/agents/ src/core/ --include='*.ts' | grep -v 'UnifiedConfigLoader' | wc -l | grep -q '^0$'
- grep -r 'UnifiedConfigLoader' src/agents/ --include='*.ts' | wc -l | awk '{if($1>=8) exit 0; else exit 1}'
- npm test -- --testPathPattern='config.*test' --coverage --coverageThreshold='{"global":{"branches":80,"functions":80,"lines":80}}'

Accept when:
- No direct process.env access exists in agent implementations outside of UnifiedConfigLoader
- All 8+ agent files reference UnifiedConfigLoader for configuration access
- Configuration validation tests achieve at least 80% code coverage and include failure scenarios

## Enforcement

- Verified by: Automated static analysis in CI pipeline checking for direct process.env access patterns
- Verified by: Code review checklist requiring configuration management review for new agent implementations
- Verified by: Integration test suite validating configuration loading behavior across all agents
- Violation handling: CI pipeline fails if direct environment variable access is detected outside approved components
- Violation handling: Pull requests introducing violations are blocked from merging until remediated
- Violation handling: Quarterly architecture reviews audit compliance and identify technical debt
- Exception process: Submit exception request to architecture team with justification and migration plan
- Exception process: Exception requires approval from at least two senior engineers
- Exception process: All exceptions are time-bound with mandatory review dates and tracked in technical debt backlog