# Standardize Configuration Loading Through Unified Environment Management: Configuration Loaders Validate

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all configuration loading, environment management, and agent initialization code paths. All modules that require runtime configuration MUST follow these patterns.

## Context

- The codebase contains multiple agent implementations (CodexCliAgent, GeminiCliAgent, MistralVibeAgent, QwenCodeAgent, RooCodeAgent, ZedAgent, OpenCodeAgent, CrushAgent) that require consistent configuration loading patterns
- Configuration sources include environment variables, file-based settings (vscode/settings.ts), MCP propagation mechanisms (propagateOpenHandsMcp.ts, propagateOpenCodeMcp.ts), and unified loaders (UnifiedConfigLoader.ts, ConfigLoader.ts)
- The pattern was detected across 17 files with 91.14% confidence, indicating a strong architectural convention for centralized configuration management
- The apply-engine.ts and mcp path modules demonstrate the need for configuration to be accessible across different execution contexts and runtime environments
- Without standardized configuration loading, agents and core modules would implement inconsistent initialization patterns leading to maintenance burden and runtime errors

## Problem Statement

How should the system load, validate, and propagate configuration data across multiple agents, execution contexts, and runtime environments while maintaining consistency, testability, and avoiding configuration drift between components?

## Decision

1. MUST: Configuration loaders MUST validate required configuration values and fail fast with clear error messages when critical configuration is missing

## Policy Block

- MUST Configuration loaders MUST validate required configuration values and fail fast with clear error messages when critical configuration is missing

In scope:
- All agent initialization code (CodexCliAgent, GeminiCliAgent, MistralVibeAgent, QwenCodeAgent, RooCodeAgent, ZedAgent, OpenCodeAgent, CrushAgent)
- Core configuration loading modules (ConfigLoader.ts, UnifiedConfigLoader.ts)
- MCP propagation mechanisms (propagateOpenHandsMcp.ts, propagateOpenCodeMcp.ts)
- Runtime environment setup and execution contexts (apply-engine.ts, mcp.ts paths)
- Settings management modules (vscode/settings.ts)

Out of scope:
- Test fixtures and mock configurations that intentionally bypass loaders for testing purposes
- Build-time configuration and compilation settings
- Development-only debugging overrides that are explicitly marked as temporary
- Third-party library initialization that requires direct environment access

Exceptions:
- EXC-001: Test code needs to inject mock configuration without going through the full loader initialization
- EXC-002: Bootstrap code that initializes the configuration system itself needs to access raw environment variables

## Rationale

- The detection of this pattern across 17 files with 91.14% confidence indicates an established architectural convention that has proven effective for managing configuration complexity
- Centralized configuration loading provides a single point of control for validation, transformation, and error handling, reducing duplication and inconsistency
- The presence of both ConfigLoader and UnifiedConfigLoader suggests an evolution toward more sophisticated configuration management, supporting the need for standardization
- Agent implementations benefit from receiving validated, typed configuration objects rather than performing their own environment parsing, reducing initialization errors and improving testability

## Consequences

Positive:
- Consistent configuration loading patterns across all agents and core modules reduce cognitive load and maintenance burden
- Centralized validation catches configuration errors early in the application lifecycle before they cause runtime failures
- Testability improves significantly as configuration can be easily mocked and injected without modifying environment state
- Configuration changes and schema evolution can be managed in a single location rather than updating scattered environment access code

Negative:
- Introduces an additional abstraction layer that developers must understand and use correctly
- May add slight initialization overhead compared to direct environment variable access
- Requires migration effort for any existing code that directly accesses environment variables
- Configuration loader becomes a critical dependency that must be carefully maintained and tested

## Alternatives

- Allow direct environment variable access throughout the codebase without centralized loaders (rejected)
  Rejected because: Leads to scattered configuration logic, inconsistent validation, difficult testing, and configuration drift between components
  When valid: Only appropriate for very small applications with minimal configuration needs
- Use dependency injection framework to provide configuration to all components (deferred)
  Rejected because: Would require significant architectural changes and introduce additional framework dependencies; current loader pattern achieves similar benefits with less complexity
  When valid: Could be reconsidered if the application grows to require full dependency injection for other reasons
- Implement configuration as a singleton global object accessible from anywhere (rejected)
  Rejected because: Global singletons make testing difficult, hide dependencies, and create implicit coupling between modules
  When valid: Not recommended for any production codebase

## Risks

- Configuration loader becomes a bottleneck or single point of failure if not properly designed and tested
  Mitigation: Implement comprehensive unit tests for all configuration loaders, use fail-fast validation, and provide clear error messages. Monitor loader initialization performance.
  Owner: Core infrastructure team
- Developers may bypass the configuration loader pattern for convenience, leading to inconsistent usage over time
  Mitigation: Enforce through code review, linting rules, and automated checks. Document the pattern clearly and provide examples. Make the loader API easy to use.
  Owner: Engineering team and code reviewers
- Configuration schema changes may break existing agents or modules if not carefully managed
  Mitigation: Version configuration schemas, maintain backward compatibility where possible, and use deprecation warnings for configuration changes. Test all agents after configuration updates.
  Owner: Architecture team

## Implementation Notes

- Start by auditing all direct process.env access in agent files and core modules, then migrate to ConfigLoader/UnifiedConfigLoader
- Ensure all configuration loaders implement consistent error handling with descriptive messages indicating which configuration values are missing or invalid
- Document the configuration schema in a central location (e.g., README or config documentation) with examples for each agent type
- Consider implementing a configuration validation layer that runs at application startup to catch errors before any agents are initialized
- Use TypeScript interfaces to define configuration shapes and leverage type checking to prevent configuration access errors

## Continuation Context


Verify commands:
- grep -r 'process\.env' src/agents/ src/core/ --include='*.ts' | grep -v ConfigLoader | grep -v UnifiedConfigLoader
- npm test -- --grep 'configuration loading'
- find src/agents -name '*.ts' -exec grep -L 'ConfigLoader\|UnifiedConfigLoader' {} \;

Accept when:
- All agent files import and use ConfigLoader or UnifiedConfigLoader for configuration access
- No direct process.env access exists in agent or core module files except within loader implementations
- Configuration loading tests pass with 100% coverage for all supported configuration sources
- All agents successfully initialize with valid configuration and fail gracefully with clear errors when configuration is invalid

## Enforcement

- Verified by: Automated linting rules that flag direct process.env access outside of configuration loader modules
- Verified by: Code review checklist items requiring configuration loader usage verification
- Verified by: CI pipeline tests that validate configuration loading behavior for all agents
- Verified by: Static analysis tools that check for configuration access patterns
- Violation handling: CI build fails if linting rules detect direct environment access in prohibited locations
- Violation handling: Code review blocks merge until configuration loading patterns are corrected
- Violation handling: Runtime warnings logged when deprecated configuration access patterns are detected
- Violation handling: Quarterly architecture reviews identify and remediate any pattern violations
- Exception process: Developer submits exception request with justification to architecture team via issue tracker
- Exception process: Architecture team reviews within 2 business days and provides written approval or alternative solution
- Exception process: Approved exceptions are documented in code comments with ticket reference and expiration date
- Exception process: All exceptions are reviewed quarterly and either renewed, remediated, or escalated