# Standardize Environment-Aware Configuration Loading with Unified Path Resolution: Configuration Loaders Support

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all agent implementations, configuration loaders, and runtime environment initialization code. All new agents and configuration management components MUST follow these patterns.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, MistralVibeAgent, ZedAgent, CodexCliAgent, AmazonQCliAgent, GeminiCliAgent, FirebenderAgent, RooCodeAgent) that require consistent configuration loading and environment management
- Configuration sources span multiple contexts including VSCode settings, MCP (Model Context Protocol) paths, unified config loaders, and environment-specific settings requiring standardized resolution
- Agent initialization requires reliable path resolution for MCP propagation (propagateOpenCodeMcp, propagateOpenHandsMcp) and runtime environment setup across different execution contexts
- The pattern appears in 17 files with 91.14% confidence, indicating a well-established architectural approach to configuration management that has proven effective across the codebase
- Configuration loading must handle multiple environments (development, production, testing) and support both file-based and programmatic configuration sources

## Problem Statement

Without a standardized approach to configuration loading and environment management, agent implementations risk inconsistent behavior across different runtime contexts, duplicated configuration logic, and fragile path resolution that breaks when deployment environments change. The system needs a unified pattern for loading configuration, resolving paths, and managing environment-specific settings that works reliably across all agent types and execution contexts.

## Decision

1. MUST: Configuration loaders MUST support environment-aware path resolution that adapts to different runtime contexts (VSCode, CLI, MCP server)

## Policy Block

- MUST Configuration loaders MUST support environment-aware path resolution that adapts to different runtime contexts (VSCode, CLI, MCP server)

In scope:
- All agent implementations (OpenCodeAgent, MistralVibeAgent, ZedAgent, CodexCliAgent, AmazonQCliAgent, GeminiCliAgent, FirebenderAgent, RooCodeAgent)
- Configuration loading utilities (UnifiedConfigLoader, ConfigLoader)
- MCP path resolution and propagation logic (mcp.ts, propagateOpenCodeMcp, propagateOpenHandsMcp)
- VSCode extension settings management (vscode/settings.ts)
- Runtime environment initialization and apply-engine configuration

Out of scope:
- Third-party library configuration that uses its own loading mechanisms
- Build-time configuration and compilation settings
- Test fixtures and mock configuration used exclusively in test suites
- Temporary or experimental agent prototypes not yet integrated into the main codebase

Exceptions:
- EXC-001: Legacy agent implementations that are scheduled for deprecation within the next release cycle
- EXC-002: Agents with specialized configuration requirements that cannot be expressed through the standard configuration schema

## Rationale

- The pattern's 91.14% confidence across 17 files demonstrates that centralized configuration loading has become the de facto standard, reducing code duplication and maintenance burden
- Unified configuration management enables consistent behavior across different agent types, making the system more predictable and easier to debug when configuration issues arise
- Standardized path resolution prevents environment-specific bugs that are difficult to reproduce and diagnose, particularly in MCP server discovery and VSCode extension contexts
- Separating configuration loading from business logic improves testability and allows configuration validation to be performed independently of agent execution

## Consequences

Positive:
- Reduced code duplication across agent implementations, with configuration loading logic centralized in 2-3 core utilities rather than scattered across 17+ files
- Improved portability and deployment flexibility, as agents can adapt to different runtime environments without code changes
- Enhanced debuggability through consistent configuration loading patterns and standardized error messages
- Easier onboarding for new agent implementations, with clear patterns to follow and reusable configuration utilities

Negative:
- Additional abstraction layer may increase complexity for simple agents that only need minimal configuration
- Centralized configuration loaders become critical dependencies that require careful maintenance and backward compatibility management
- Performance overhead from environment detection and path resolution, though typically negligible compared to agent execution time
- Migration effort required for any legacy agents still using custom configuration loading approaches

## Alternatives

- Allow each agent to implement custom configuration loading tailored to its specific needs (rejected)
  Rejected because: Leads to code duplication, inconsistent behavior across agents, and increased maintenance burden. The pattern detection shows 17 files already converged on unified configuration, indicating custom approaches are being phased out.
  When valid: Only for experimental prototypes or agents with truly unique configuration requirements that cannot be expressed through standard schema
- Use a dependency injection framework to provide configuration to agents at runtime (deferred)
  Rejected because: Would require significant refactoring of existing agent implementations and introduce additional framework dependencies. May be reconsidered if the number of agents grows significantly.
  When valid: If the codebase scales to 50+ agent types or requires complex configuration composition and override scenarios
- Implement configuration as environment variables only, eliminating file-based configuration (rejected)
  Rejected because: Environment variables alone are insufficient for complex configuration structures, difficult to manage for local development, and don't support hierarchical configuration or type validation
  When valid: For containerized deployments where 12-factor app principles are strictly enforced and configuration is simple

## Risks

- Breaking changes to UnifiedConfigLoader or ConfigLoader could impact all 17+ agent implementations simultaneously
  Mitigation: Implement comprehensive integration tests covering all agent types, use semantic versioning for configuration loader APIs, and maintain backward compatibility for at least one major version
  Owner: Platform Engineering Team
- Environment-specific path resolution may fail in unexpected deployment contexts (containers, serverless, edge environments)
  Mitigation: Provide explicit configuration override mechanisms, implement fallback strategies for path resolution, and document supported deployment environments with examples
  Owner: DevOps and Agent Development Teams
- Configuration loading errors may be silently ignored or produce unclear error messages, leading to runtime failures
  Mitigation: Implement strict validation with fail-fast behavior, provide detailed error messages with remediation guidance, and add configuration validation to CI/CD pipeline
  Owner: Quality Engineering Team

## Implementation Notes

- New agent implementations should extend a base agent class that handles configuration loading, or use dependency injection to receive a pre-configured ConfigLoader instance
- Configuration loaders should validate schema on load and throw descriptive errors for missing required fields, invalid types, or malformed paths
- MCP path resolution should use the standardized utilities in mcp.ts to ensure consistent server discovery across VSCode, CLI, and programmatic contexts
- Consider implementing a configuration dry-run mode that validates configuration without executing agent logic, useful for deployment validation and troubleshooting
- Document the configuration precedence order (explicit params > env vars > config files > defaults) in both code comments and user-facing documentation

## Continuation Context


Verify commands:
- grep -r 'new UnifiedConfigLoader\|new ConfigLoader\|import.*ConfigLoader' src/agents/ | wc -l
- grep -r 'process\.env\.' src/agents/ | grep -v ConfigLoader | wc -l
- find src/agents -name '*.ts' -exec grep -L 'ConfigLoader\|UnifiedConfigLoader' {} \; | wc -l

Accept when:
- All agent implementations in src/agents/ import and use either UnifiedConfigLoader or ConfigLoader rather than implementing custom configuration loading
- Direct process.env access in agent files is minimal (< 5 occurrences) and only for non-configuration purposes
- Fewer than 3 agent implementation files lack any reference to standardized configuration loaders

## Enforcement

- Verified by: Automated code review checks in CI pipeline scanning for direct process.env usage and custom configuration loading patterns
- Verified by: Architecture review for all new agent implementations verifying use of standardized configuration loaders
- Verified by: Integration tests validating that all agents can load configuration from standard sources (files, env vars, explicit params)
- Violation handling: CI pipeline fails if new agent implementations bypass UnifiedConfigLoader/ConfigLoader without documented exception
- Violation handling: Code review requires explicit justification and architecture team approval for any custom configuration loading
- Violation handling: Quarterly audits identify agents not following the pattern and create migration tickets with priority based on maintenance burden
- Exception process: Submit exception request to architecture review board with detailed justification explaining why standard configuration is insufficient
- Exception process: Provide alternative design document showing how custom configuration loading will be maintained and tested
- Exception process: Obtain approval from architecture team lead and document exception in agent source file header with expiration date for review