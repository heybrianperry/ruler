# Standardize Configuration Loading with Unified Environment Management: Configuration Loading Use

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all agent implementations, configuration loaders, and runtime environment management components. All code that loads configuration, manages environment variables, or initializes runtime settings MUST comply with these rules.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, MistralVibeAgent, ZedAgent, CodexCliAgent, AmazonQCliAgent, GeminiCliAgent, FirebenderAgent, RooCodeAgent) that require consistent configuration loading patterns
- Configuration sources include TOML files, environment variables, VSCode settings, and MCP (Model Context Protocol) propagation mechanisms, creating complexity in managing runtime environments
- A UnifiedConfigLoader component exists to centralize configuration management, indicating a deliberate architectural decision to standardize configuration access patterns
- The security.input_validation facet indicates that configuration values require validation and sanitization before use to prevent injection attacks and runtime errors
- Multiple configuration propagation mechanisms (propagateOpenCodeMcp, propagateOpenHandsMcp) suggest a need for consistent configuration distribution across distributed agent systems

## Problem Statement

Without a standardized approach to configuration loading and environment management, the system faces risks of inconsistent behavior across agents, security vulnerabilities from unvalidated configuration inputs, and maintenance complexity from duplicated configuration logic. The proliferation of agent implementations requires a unified pattern that ensures all configuration sources are validated, properly scoped, and consistently accessed across the runtime environment.

## Decision

1. MUST: All configuration loading MUST use the UnifiedConfigLoader component or an equivalent centralized configuration management system

## Policy Block

- MUST All configuration loading MUST use the UnifiedConfigLoader component or an equivalent centralized configuration management system

In scope:
- All agent implementations (OpenCodeAgent, MistralVibeAgent, ZedAgent, CodexCliAgent, AmazonQCliAgent, GeminiCliAgent, FirebenderAgent, RooCodeAgent)
- Configuration loader components (UnifiedConfigLoader, settings loaders)
- MCP propagation mechanisms (propagateOpenCodeMcp, propagateOpenHandsMcp)
- Runtime environment initialization code
- VSCode extension settings management
- TOML and other configuration file parsers

Out of scope:
- Build-time configuration (webpack, tsconfig)
- Test fixtures with hardcoded configuration values
- Development-only debugging code that temporarily bypasses configuration for troubleshooting
- Third-party library configuration that doesn't integrate with the application configuration system

Exceptions:
- EXC-001: Emergency hotfix requires direct environment variable access to bypass a broken configuration loader
- EXC-002: Legacy agent implementations during migration period before UnifiedConfigLoader adoption is complete

## Rationale

- The pattern appears in 18 files with 91.07% confidence, indicating a strong, consistent architectural decision across the codebase
- Centralized configuration management reduces code duplication, improves maintainability, and provides a single point for implementing security controls like input validation
- The security.input_validation facet indicates that configuration values are a potential attack vector; standardizing validation at the configuration layer provides defense-in-depth
- Multiple agent implementations benefit from consistent configuration patterns, reducing cognitive load for developers and ensuring predictable behavior across the system

## Consequences

Positive:
- Consistent configuration behavior across all agents and components reduces debugging time and prevents configuration-related bugs
- Centralized validation provides a single point to implement security controls, reducing the attack surface for configuration injection vulnerabilities
- Type-safe configuration interfaces catch configuration errors at compile time rather than runtime
- Easier to add new configuration sources or modify precedence rules without touching individual agent implementations

Negative:
- Introduces a dependency on the UnifiedConfigLoader component, creating a potential single point of failure
- May add slight performance overhead from validation and abstraction layers compared to direct environment variable access
- Requires migration effort for existing code that directly accesses configuration sources
- Learning curve for developers who need to understand the configuration abstraction layer rather than simple environment variable access

## Alternatives

- Allow each agent to implement its own configuration loading logic with direct environment variable access (rejected)
  Rejected because: Creates inconsistent behavior across agents, duplicates validation logic, and increases security risk from unvalidated inputs
  When valid: Only appropriate for simple single-agent systems with minimal configuration requirements
- Use a third-party configuration management library (e.g., dotenv, config, convict) without custom abstraction (rejected)
  Rejected because: Third-party libraries may not support all required configuration sources (TOML, MCP, VSCode settings) and lack domain-specific validation logic
  When valid: Could be used as the underlying implementation within UnifiedConfigLoader
- Implement configuration as a dependency injection container with explicit configuration objects passed to constructors (deferred)
  Rejected because: Not rejected, but deferred for future consideration as it requires significant architectural refactoring
  When valid: Could be adopted in a future major version as part of broader dependency injection adoption

## Risks

- UnifiedConfigLoader becomes a bottleneck or single point of failure if not properly designed for reliability
  Mitigation: Implement comprehensive error handling, fallback mechanisms, and extensive testing of the configuration loader. Consider circuit breaker patterns for external configuration sources.
  Owner: Platform Engineering Team
- Migration from direct configuration access to UnifiedConfigLoader may introduce regressions in existing agent behavior
  Mitigation: Implement feature flags to gradually roll out configuration changes, maintain backward compatibility during migration, and add integration tests comparing old vs new configuration behavior
  Owner: Agent Development Team
- Performance degradation from validation overhead in high-frequency configuration access scenarios
  Mitigation: Implement caching for validated configuration values, lazy validation where appropriate, and performance benchmarks to monitor configuration access latency
  Owner: Performance Engineering Team

## Implementation Notes

- Start by auditing all direct process.env accesses and configuration file reads across the codebase to identify migration candidates
- Define TypeScript interfaces or Zod schemas for all configuration objects to enable compile-time type checking
- Implement UnifiedConfigLoader with support for configuration source precedence: CLI args > environment variables > config files > defaults
- Add comprehensive logging (at debug level) for configuration loading to aid troubleshooting, ensuring sensitive values are redacted
- Create migration guides and examples for agent developers showing how to convert from direct configuration access to UnifiedConfigLoader
- Consider implementing a configuration validation CLI tool that can check configuration files before deployment

## Continuation Context


Verify commands:
- grep -r 'process\.env\[' src/ --exclude-dir=node_modules | grep -v UnifiedConfigLoader | wc -l
- npm run test:config -- --grep 'configuration validation'
- eslint src/ --rule 'no-restricted-syntax: [error, {selector: MemberExpression[object.object.name=process][object.property.name=env], message: Use UnifiedConfigLoader instead of direct process.env access}]'

Accept when:
- All agent implementations load configuration exclusively through UnifiedConfigLoader or approved abstraction layer
- Configuration validation tests pass for all supported configuration sources (TOML, environment variables, VSCode settings, MCP)
- No direct process.env accesses exist outside of the UnifiedConfigLoader implementation itself
- All configuration schemas are defined with type-safe interfaces and validation logic

## Enforcement

- Verified by: ESLint rules preventing direct process.env access outside approved configuration modules
- Verified by: Code review checklist requiring configuration changes to use UnifiedConfigLoader
- Verified by: CI pipeline integration tests validating configuration loading behavior
- Verified by: Static analysis tools checking for configuration validation on all external inputs
- Violation handling: ESLint violations block CI pipeline and prevent merge
- Violation handling: Code review identifies violations and requests changes before approval
- Violation handling: Security scanning tools flag unvalidated configuration access as medium-severity findings
- Violation handling: Quarterly architecture reviews audit configuration patterns and identify non-compliant code
- Exception process: Developer submits exception request to architecture team with justification and impact analysis
- Exception process: Architecture team reviews within 2 business days and approves/rejects with written rationale
- Exception process: Approved exceptions are documented in code with TODO comments and tracking issues
- Exception process: All exceptions are reviewed quarterly and must be re-justified or remediated