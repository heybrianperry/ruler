# Adopt Unified Configuration Loading Pattern for Runtime Environment Management: Configuration Loading Logic

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all agent implementations and configuration management components within the system. All new agents and configuration loaders MUST comply with these rules.

## Context

- The system contains multiple agent implementations (ZedAgent, GeminiCliAgent, RooCodeAgent, OpenCodeAgent, QwenCodeAgent) that require consistent configuration loading mechanisms
- Configuration sources include TOML files, environment variables, and runtime parameters that must be unified across different execution contexts
- A UnifiedConfigLoader component has been established as the central pattern for managing configuration across the codebase
- The apply-engine and MCP propagation components demonstrate the need for standardized configuration access patterns to ensure consistent behavior
- Testing infrastructure (test_manual_toml.js, test_toml_options.js) validates configuration loading behavior, indicating this is a critical architectural concern

## Problem Statement

Without a standardized configuration loading pattern, each agent and component may implement its own configuration management approach, leading to inconsistent behavior, duplicated code, difficult testing, and potential configuration conflicts. The system needs a unified approach to load, validate, and access configuration data from multiple sources (TOML files, environment variables, CLI arguments) while maintaining type safety and predictable precedence rules.

## Decision

1. MUST: Configuration loading logic MUST be testable in isolation with test fixtures for TOML files and environment variable scenarios

## Policy Block

- MUST Configuration loading logic MUST be testable in isolation with test fixtures for TOML files and environment variable scenarios

In scope:
- All agent implementations (ZedAgent, GeminiCliAgent, RooCodeAgent, OpenCodeAgent, QwenCodeAgent, and future agents)
- Core configuration management components (UnifiedConfigLoader, apply-engine)
- MCP propagation and integration components that require configuration access
- Test infrastructure that validates configuration loading behavior

Out of scope:
- Third-party library configuration that uses library-specific configuration mechanisms
- Build-time configuration that is resolved during compilation or bundling
- Temporary or ephemeral configuration used only within a single function scope
- Configuration for external services that have their own configuration management systems

Exceptions:
- EXC-001: A component requires configuration before UnifiedConfigLoader is initialized (e.g., bootstrap configuration)
- EXC-002: Performance-critical paths where configuration loading overhead is measured and documented as unacceptable

## Rationale

- Pattern detected across 10 files with 90.96% confidence indicates this is an established architectural standard in the codebase
- Multiple agent implementations following the same pattern demonstrates intentional design rather than coincidental similarity
- The presence of dedicated test files (test_manual_toml.js, test_toml_options.js) indicates this pattern is considered critical enough to warrant comprehensive testing
- Centralized configuration management reduces code duplication, improves maintainability, and ensures consistent behavior across all agents and components

## Consequences

Positive:
- Consistent configuration behavior across all agents and components reduces debugging time and cognitive load
- Centralized configuration loading logic makes it easier to add new configuration sources or modify precedence rules
- Type-safe configuration validation catches errors early in the development cycle rather than at runtime
- Testable configuration loading enables comprehensive test coverage for different configuration scenarios

Negative:
- Introduces a dependency on UnifiedConfigLoader that all components must use, creating a potential single point of failure
- May add slight overhead for simple components that only need minimal configuration
- Requires developers to learn and follow the UnifiedConfigLoader API rather than using simpler direct file reading
- Configuration precedence rules may become complex when multiple sources provide conflicting values

## Alternatives

- Allow each agent to implement its own configuration loading logic with no standardization (rejected)
  Rejected because: Leads to code duplication, inconsistent behavior, and makes it difficult to change configuration sources or precedence rules globally
  When valid: Never valid for this codebase given the number of agents requiring consistent configuration
- Use a dependency injection framework to provide configuration to components (rejected)
  Rejected because: Adds significant complexity and external dependencies for a problem that can be solved with a simpler centralized loader pattern
  When valid: Could be reconsidered if the system grows to require comprehensive dependency injection for other concerns beyond configuration
- Use environment variables exclusively and eliminate TOML configuration files (rejected)
  Rejected because: TOML files provide better structure for complex configuration, version control integration, and human readability compared to flat environment variables
  When valid: May be valid for containerized deployments where environment variables are the standard, but should still use UnifiedConfigLoader to abstract the source

## Risks

- UnifiedConfigLoader becomes a bottleneck or single point of failure if it has bugs or performance issues
  Mitigation: Maintain comprehensive test coverage for UnifiedConfigLoader, implement error handling with clear error messages, and monitor performance in production
  Owner: Engineering team
- Configuration precedence rules may be unclear to developers, leading to unexpected behavior when multiple sources provide values
  Mitigation: Document precedence rules clearly, provide logging/debugging output showing which configuration source was used for each value, and include examples in documentation
  Owner: Engineering team
- Schema validation for TOML files may become outdated as configuration requirements evolve
  Mitigation: Implement automated schema generation from TypeScript types, include schema validation in CI pipeline, and maintain schema documentation alongside code
  Owner: Engineering team

## Implementation Notes

- Import UnifiedConfigLoader at the entry point of your application or agent and load configuration before instantiating agents
- Define TypeScript interfaces for your configuration structure and use them to type the configuration objects returned by UnifiedConfigLoader
- When adding new configuration options, update the TOML schema validation rules and add test cases covering the new options
- Use environment variable prefixes (e.g., APP_) to avoid conflicts with system environment variables
- Consider providing a configuration validation command that can be run independently to check TOML files before deployment

## Continuation Context


Verify commands:
- grep -r "UnifiedConfigLoader" src/agents/ | wc -l
- grep -r "import.*UnifiedConfigLoader" src/ --include="*.ts" --include="*.js"
- npm test -- --grep "config.*load" 2>&1 | grep -E "(passing|failing)"

Accept when:
- All agent implementations import and use UnifiedConfigLoader for configuration loading
- Configuration loading tests pass successfully, including tests for TOML parsing, environment variable precedence, and validation
- No direct file system reads for configuration files exist outside of UnifiedConfigLoader implementation

## Enforcement

- Verified by: Code review process checks that new agents use UnifiedConfigLoader
- Verified by: CI pipeline runs configuration loading tests on every commit
- Verified by: Static analysis tools can be configured to flag direct configuration file reads outside UnifiedConfigLoader
- Violation handling: Pull requests that bypass UnifiedConfigLoader are rejected during code review with feedback to use the standard pattern
- Violation handling: Existing violations are tracked as technical debt items and prioritized for refactoring
- Violation handling: Documentation and onboarding materials emphasize the UnifiedConfigLoader pattern to prevent new violations
- Exception process: Developer identifies a legitimate need to bypass UnifiedConfigLoader and documents the rationale
- Exception process: Exception request is submitted to architecture team with performance data or technical justification
- Exception process: If approved, exception is documented in code comments with reference to the exception ID and approval
- Exception process: Exceptions are reviewed quarterly to determine if they can be eliminated through improvements to UnifiedConfigLoader