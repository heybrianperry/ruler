# Standardize Environment Variable Access for Configuration Management: Configuration Values That

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all configuration loading, agent initialization, and runtime environment setup code.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, MistralVibeAgent, ZedAgent, CodexCliAgent, AmazonQCliAgent, GeminiCliAgent, FirebenderAgent, RooCodeAgent) that require runtime configuration from environment variables
- Configuration loading is centralized through ConfigLoader and UnifiedConfigLoader components that must consistently access environment variables across different execution contexts
- MCP (Model Context Protocol) propagation modules (propagateOpenCodeMcp, propagateOpenHandsMcp) need to pass environment configuration to child processes and external tools
- VSCode extension settings integration requires mapping between editor configuration and environment variables for seamless developer experience
- The pattern appears in 17 files with 91.14% confidence, indicating a well-established architectural convention for environment-based configuration

## Problem Statement

Without a standardized approach to accessing environment variables for configuration management, the system risks inconsistent configuration behavior across agents, difficulty in testing and mocking configurations, unclear precedence rules when multiple configuration sources exist, and challenges in debugging configuration-related issues in production environments.

## Decision

1. MUST: All configuration values that vary by deployment environment MUST be sourced from environment variables using process.env

## Policy Block

- MUST All configuration values that vary by deployment environment MUST be sourced from environment variables using process.env

In scope:
- All agent implementations (OpenCodeAgent, MistralVibeAgent, ZedAgent, CodexCliAgent, AmazonQCliAgent, GeminiCliAgent, FirebenderAgent, RooCodeAgent)
- Configuration loading modules (ConfigLoader, UnifiedConfigLoader)
- MCP propagation modules that pass configuration to child processes
- VSCode extension settings integration code
- Runtime environment setup and initialization code

Out of scope:
- Build-time configuration and constants that never change across environments
- Type definitions and interfaces that don't access runtime values
- Pure utility functions that don't depend on configuration
- Test fixtures and mocks that intentionally bypass the configuration system

Exceptions:
- EXC-001: Legacy code during migration period where refactoring to use ConfigLoader would introduce unacceptable risk
- EXC-002: Bootstrap code that initializes the configuration loader itself

## Rationale

- Pattern detected across 17 files with 91.14% confidence indicates this is an established architectural convention that has proven effective in practice
- Centralized configuration management through ConfigLoader/UnifiedConfigLoader provides a single point of control for environment variable access, enabling consistent behavior, easier testing through mocking, and simplified debugging
- Agent implementations require consistent access to runtime configuration for API keys, endpoints, feature flags, and operational parameters that must vary across development, staging, and production environments
- MCP propagation requirements demonstrate the need for configuration to be explicitly passed to child processes, which is best managed through a centralized configuration abstraction rather than ad-hoc environment variable access

## Consequences

Positive:
- Improved testability through ability to mock configuration loader rather than manipulating process.env directly
- Consistent configuration behavior across all agents and components reduces environment-specific bugs
- Centralized validation and error handling for missing or invalid configuration values improves developer experience
- Clear separation between configuration concerns and business logic improves code maintainability and readability
- Easier to implement configuration precedence rules (env vars > config files > defaults) in a single location

Negative:
- Additional abstraction layer adds slight complexity and indirection when accessing configuration values
- Requires discipline to avoid bypassing the configuration loader with direct process.env access
- Migration of existing code that directly accesses process.env requires refactoring effort
- Configuration loader becomes a critical dependency that must be carefully maintained and tested

## Alternatives

- Allow direct process.env access throughout the codebase without a configuration loader abstraction (rejected)
  Rejected because: Direct process.env access scattered throughout the codebase makes testing difficult, provides no validation or error handling, and creates inconsistent configuration behavior across components
  When valid: Only appropriate for very small applications with minimal configuration needs and no testing requirements
- Use a third-party configuration management library (e.g., dotenv, config, convict) (rejected)
  Rejected because: The existing ConfigLoader/UnifiedConfigLoader pattern is already established across 17 files and provides the necessary functionality; introducing a third-party library would require significant migration effort without clear benefits
  When valid: Could be reconsidered if the custom configuration loader becomes too complex to maintain or if advanced features like schema validation and type safety are required
- Use dependency injection to pass configuration values explicitly to each component (deferred)
  Rejected because: Not rejected but deferred; could be complementary to the configuration loader pattern for improved testability and explicit dependencies
  When valid: Could be adopted incrementally for new components while maintaining the configuration loader for backward compatibility

## Risks

- Configuration loader becomes a single point of failure; bugs in the loader affect all components
  Mitigation: Implement comprehensive unit tests for configuration loader; keep loader logic simple and focused; conduct thorough code review for any changes to configuration loading code
  Owner: Engineering team
- Developers may bypass the configuration loader with direct process.env access, undermining consistency
  Mitigation: Establish linting rules to detect direct process.env access outside approved locations; include configuration patterns in code review checklist; provide clear documentation and examples
  Owner: Engineering team
- Configuration precedence rules (env vars vs config files vs defaults) may become complex and difficult to understand
  Mitigation: Document precedence rules clearly; provide diagnostic logging that shows which configuration source was used for each value; implement a configuration debug mode that displays all active configuration
  Owner: Engineering team

## Implementation Notes

- Create or enhance ConfigLoader/UnifiedConfigLoader with methods for each configuration category (agent config, MCP config, VSCode settings, etc.)
- Implement a configuration validation layer that checks required environment variables at application startup and provides clear error messages with examples
- Add logging to configuration loader to track which configuration sources are being used (useful for debugging production issues)
- Provide a .env.example file documenting all supported environment variables with descriptions and example values
- Consider implementing a configuration schema with TypeScript types to provide compile-time safety for configuration access

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' src/ --exclude-dir=node_modules | grep -v 'ConfigLoader\|UnifiedConfigLoader' | wc -l
- npm test -- --testPathPattern='ConfigLoader|UnifiedConfigLoader' --coverage
- grep -r 'new.*Agent' src/agents/ | xargs grep -L 'ConfigLoader\|config'

Accept when:
- First verify command returns 0 or only shows approved exceptions (bootstrap code in ConfigLoader itself)
- Configuration loader tests achieve >90% code coverage and include tests for missing required variables, default values, and precedence rules
- All agent implementations retrieve configuration through ConfigLoader/UnifiedConfigLoader rather than direct environment variable access

## Enforcement

- Verified by: Automated linting rules that flag direct process.env access outside approved files
- Verified by: Code review checklist includes verification that new configuration access uses ConfigLoader
- Verified by: CI pipeline runs verify commands and fails if direct process.env usage is detected in non-approved locations
- Verified by: Unit test coverage requirements for configuration-dependent code
- Violation handling: CI build fails if linting rules detect unapproved direct process.env access
- Violation handling: Code review blocks merge if configuration access doesn't follow the pattern
- Violation handling: Existing violations are tracked as technical debt with migration tickets
- Violation handling: Quarterly architecture review audits configuration access patterns
- Exception process: Developer documents the exception reason and creates a ticket for the exception request
- Exception process: Architecture review board evaluates the exception request within 2 business days
- Exception process: Approved exceptions are documented in code with TODO comments including ticket reference
- Exception process: All exceptions include a target date for migration to the standard pattern