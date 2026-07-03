# Adopt Unified Configuration Loading with Environment Variable Override Support: Configuration Loading Behavior

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all configuration loading operations across agents and core systems.

## Context

- The codebase contains multiple agents (OpenCodeAgent, MistralVibeAgent) and core configuration systems (UnifiedConfigLoader) that require consistent configuration management across different runtime environments
- Configuration sources include TOML files, environment variables, and programmatic options, requiring a unified approach to merge and prioritize these sources
- Test files (test_manual_toml.js, test_toml_options.js) demonstrate explicit testing of configuration loading behavior, indicating this is a critical architectural concern
- The pattern appears in 5 files with 90.72% confidence, suggesting a well-established architectural practice rather than an ad-hoc implementation
- Runtime environment flexibility is essential for supporting development, testing, staging, and production deployments with different configuration needs

## Problem Statement

Systems require a consistent, predictable mechanism for loading configuration from multiple sources (files, environment variables, programmatic options) with clear precedence rules, while maintaining testability and supporting diverse runtime environments without code duplication or configuration drift.

## Decision

1. MUST: Configuration loading behavior MUST be covered by automated tests that verify precedence rules and override mechanisms

## Policy Block

- MUST Configuration loading behavior MUST be covered by automated tests that verify precedence rules and override mechanisms

In scope:
- All agent implementations (OpenCodeAgent, MistralVibeAgent, and future agents)
- Core configuration systems (UnifiedConfigLoader and related utilities)
- Application startup and initialization code
- Test suites that verify configuration behavior
- Deployment scripts and environment setup procedures

Out of scope:
- Third-party library configuration that uses proprietary formats
- Temporary or ephemeral runtime state that is not persisted
- Hard-coded constants that are not intended to be configurable
- Legacy systems scheduled for deprecation within the next release cycle

Exceptions:
- EXC-001: A third-party integration requires a specific configuration format that cannot be adapted to the unified loader
- EXC-002: Performance-critical paths require direct configuration access to avoid loader overhead

## Rationale

- Pattern detected across 5 files with 90.72% confidence indicates this is an established architectural practice that has proven effective
- Unified configuration loading reduces code duplication, prevents configuration drift, and ensures consistent behavior across all system components
- Environment variable override support enables 12-factor app compliance and simplifies deployment across different environments without code changes
- Explicit test coverage (test_manual_toml.js, test_toml_options.js) demonstrates that configuration loading is a critical path requiring reliability guarantees

## Consequences

Positive:
- Consistent configuration behavior across all agents and core systems reduces debugging time and operational errors
- Environment variable overrides enable seamless deployment across development, staging, and production without configuration file modifications
- Centralized configuration loading logic simplifies maintenance and enables global improvements (validation, schema enforcement, hot-reloading)
- Test coverage of configuration loading provides confidence in deployment and reduces configuration-related production incidents

Negative:
- Unified configuration loader becomes a critical dependency that all components rely on, creating a potential single point of failure
- Configuration precedence rules add complexity that developers must understand to avoid unexpected behavior
- TOML format requirement may create friction for teams familiar with JSON or YAML configuration formats
- Additional abstraction layer may introduce slight performance overhead compared to direct configuration access

## Alternatives

- Allow each agent to implement custom configuration loading tailored to its specific needs (rejected)
  Rejected because: Creates configuration drift, duplicates code, and makes it difficult to ensure consistent behavior across environments. Testing becomes more complex as each agent requires separate configuration test suites.
  When valid: Only valid for prototype or experimental agents that are explicitly marked as non-production
- Use JSON format as the primary configuration file format instead of TOML (rejected)
  Rejected because: TOML provides better human readability for configuration files, supports comments, and has stronger typing semantics. The pattern evidence shows TOML is already established in the codebase.
  When valid: Could be reconsidered if TOML tooling proves inadequate or if JSON Schema validation becomes a critical requirement
- Use a configuration management service (e.g., Consul, etcd) for centralized configuration (deferred)
  Rejected because: Adds operational complexity and external dependencies. Current file-based + environment variable approach is sufficient for current scale.
  When valid: Should be reconsidered when deploying to large-scale distributed systems requiring dynamic configuration updates

## Risks

- UnifiedConfigLoader becomes a bottleneck or single point of failure if not properly designed and tested
  Mitigation: Maintain comprehensive test coverage (as evidenced by test_manual_toml.js and test_toml_options.js), implement fail-fast validation, and provide clear error messages. Consider circuit breaker patterns for external configuration sources.
  Owner: Engineering team (core infrastructure)
- Configuration precedence rules may be misunderstood, leading to unexpected behavior in production
  Mitigation: Document precedence rules clearly, provide configuration debugging tools that show effective configuration values and their sources, and include precedence validation in test suites.
  Owner: Engineering team (documentation and tooling)
- Sensitive configuration values (API keys, secrets) may be inadvertently logged or exposed through configuration debugging tools
  Mitigation: Implement secret detection and masking in configuration logging, use dedicated secret management for sensitive values, and audit configuration access patterns.
  Owner: Security team

## Implementation Notes

- Ensure UnifiedConfigLoader provides clear error messages that indicate which configuration source failed and why (e.g., 'TOML parse error in config.toml line 15' or 'Missing required environment variable: API_KEY')
- Implement a configuration debugging mode that outputs the effective configuration and the source of each value (file, env var, default) to aid troubleshooting
- Document the naming convention for environment variables (e.g., APP_SECTION_KEY format) and provide examples for common configuration scenarios
- Create migration guides for any existing agents that use custom configuration loading to help teams adopt the unified approach
- Consider implementing configuration validation schemas (e.g., using JSON Schema or similar) to catch configuration errors at startup rather than runtime

## Continuation Context


Verify commands:
- grep -r 'UnifiedConfigLoader\|ConfigLoader' --include='*.ts' --include='*.js' src/ | wc -l
- grep -r 'process\.env\.' --include='*.ts' --include='*.js' src/agents/ | grep -v 'UnifiedConfigLoader' | wc -l
- find test/ -name '*toml*.js' -o -name '*config*.test.ts' | xargs grep -l 'precedence\|override\|environment' | wc -l

Accept when:
- All agents use UnifiedConfigLoader (grep shows consistent usage across agent files)
- No direct process.env access in agent implementations that bypasses the unified loader (second grep returns 0 or minimal results)
- Configuration loading tests exist that verify precedence and override behavior (third command shows test files covering these scenarios)

## Enforcement

- Verified by: Automated code review checks that flag direct process.env access in agent implementations
- Verified by: CI pipeline runs configuration loading tests (test_manual_toml.js, test_toml_options.js) on every commit
- Verified by: Architecture review for new agents includes verification of UnifiedConfigLoader usage
- Verified by: Static analysis tools detect configuration loading patterns that bypass the unified loader
- Violation handling: CI build fails if configuration loading tests do not pass
- Violation handling: Code review blocks merge if direct configuration access bypasses UnifiedConfigLoader without documented exception
- Violation handling: Runtime warnings logged if configuration is accessed through non-standard mechanisms
- Violation handling: Quarterly audits identify and remediate configuration loading anti-patterns
- Exception process: Submit exception request to architecture review board with justification (performance, third-party integration, etc.)
- Exception process: Provide benchmarks or technical constraints that prevent use of UnifiedConfigLoader
- Exception process: Document exception in code comments and architecture decision log
- Exception process: Include migration plan or timeline for bringing exceptional cases into compliance if possible