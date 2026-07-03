# Adopt Unified Configuration Loading with Input Validation: Components Extend Unifiedconfigloader

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all configuration loading operations across agent implementations, core engine components, and test suites. All configuration sources MUST be processed through the unified configuration loader with mandatory input validation.

## Context

- The codebase contains multiple agent implementations (ZedAgent, GeminiCliAgent, RooCodeAgent, OpenCodeAgent, QwenCodeAgent) that require consistent configuration loading mechanisms
- Configuration sources include TOML files, environment variables, and runtime parameters that need unified handling across the application
- The UnifiedConfigLoader pattern emerged as a centralized approach to handle configuration loading with consistent validation and error handling
- Security concerns around input validation necessitate a standardized approach to prevent configuration injection attacks and malformed data processing
- The apply-engine and MCP propagation components require reliable configuration management to ensure consistent runtime behavior

## Problem Statement

Without a unified configuration loading mechanism, each component implements its own configuration parsing logic, leading to inconsistent validation, duplicated code, security vulnerabilities from unvalidated inputs, and maintenance overhead. The system needs a standardized approach to load, validate, and manage configuration data across all agents and core components while ensuring security through proper input validation.

## Decision

1. MAY: Components MAY extend the UnifiedConfigLoader with domain-specific validation rules when necessary

## Policy Block

- MAY Components MAY extend the UnifiedConfigLoader with domain-specific validation rules when necessary

In scope:
- All agent implementations (ZedAgent, GeminiCliAgent, RooCodeAgent, OpenCodeAgent, QwenCodeAgent)
- Core engine components (apply-engine, configuration loaders)
- MCP propagation modules
- Test suites that exercise configuration loading (test_manual_toml.js, test_toml_options.js)
- Any component that reads TOML files, environment variables, or runtime configuration

Out of scope:
- Hard-coded constants that do not vary by environment
- Compile-time configuration embedded in source code
- Internal data structures that are not externally configurable
- Temporary test fixtures with inline configuration for unit tests

Exceptions:
- EXC-001: Legacy components during migration phase may temporarily use direct configuration parsing
- EXC-002: Performance-critical paths where configuration is loaded once at startup and cached

## Rationale

- Pattern detected across 10 files with 90.96% confidence indicates strong architectural consistency around unified configuration loading
- Security facet (input_validation) demonstrates intentional design to prevent configuration-based vulnerabilities and injection attacks
- Multiple agent implementations sharing the same pattern suggests successful reuse and proven approach
- Centralized configuration management reduces code duplication and ensures consistent behavior across the application

## Consequences

Positive:
- Consistent configuration validation across all components reduces security vulnerabilities
- Centralized configuration loading reduces code duplication and maintenance overhead
- Standardized error handling improves debugging and troubleshooting experience
- Easier to add new configuration sources or formats in the future
- Improved testability through unified mocking and fixture management

Negative:
- Additional abstraction layer may introduce slight performance overhead for configuration loading
- Teams must learn and adopt the UnifiedConfigLoader API rather than using familiar direct parsing
- Migration effort required for existing components using custom configuration logic
- Potential single point of failure if UnifiedConfigLoader has bugs or limitations

## Alternatives

- Allow each component to implement its own configuration loading with shared validation utilities (rejected)
  Rejected because: Leads to inconsistent implementation, duplicated code, and higher risk of validation gaps across components
  When valid: Only appropriate for components with truly unique configuration requirements that cannot be generalized
- Use a third-party configuration management library (e.g., convict, config, dotenv) (rejected)
  Rejected because: External dependencies may not provide the specific validation and security requirements needed; custom UnifiedConfigLoader offers better control
  When valid: Could be reconsidered if the custom solution becomes too complex to maintain or lacks critical features
- Implement configuration as code using TypeScript modules with type safety (rejected)
  Rejected because: Requires recompilation for configuration changes and doesn't support runtime configuration updates or environment-specific overrides
  When valid: Suitable for truly static configuration that never varies across environments

## Risks

- UnifiedConfigLoader becomes a bottleneck or single point of failure if not properly designed
  Mitigation: Implement comprehensive test coverage, error handling, and fallback mechanisms. Monitor performance and reliability metrics.
  Owner: Engineering team
- Migration from existing configuration approaches may introduce bugs or breaking changes
  Mitigation: Implement gradual migration with parallel validation, comprehensive testing, and rollback capability. Maintain backward compatibility where possible.
  Owner: Engineering team
- Overly strict validation may reject valid edge-case configurations
  Mitigation: Design validation rules to be configurable and extensible. Provide clear error messages and documentation for validation requirements. Implement exception handling process.
  Owner: Engineering team

## Implementation Notes

- Start by auditing all existing configuration loading code to identify patterns and requirements that the UnifiedConfigLoader must support
- Implement the UnifiedConfigLoader with a plugin architecture to allow domain-specific validation extensions while maintaining core consistency
- Create migration guides and examples for each agent type to facilitate adoption across the codebase
- Establish comprehensive test suites (like test_manual_toml.js and test_toml_options.js) that verify both valid and invalid configuration scenarios
- Document the configuration schema and validation rules clearly for developers and operators

## Continuation Context


Verify commands:
- grep -r 'UnifiedConfigLoader' src/ | wc -l
- grep -r 'new UnifiedConfigLoader\|import.*UnifiedConfigLoader' src/agents/ src/core/ src/mcp/
- npm test -- --grep 'configuration.*validation'
- eslint src/ --rule 'no-restricted-imports: [error, {patterns: ["**/direct-config-parser"]}]'

Accept when:
- All agent implementations (ZedAgent, GeminiCliAgent, RooCodeAgent, OpenCodeAgent, QwenCodeAgent) import and use UnifiedConfigLoader
- Configuration validation tests pass for both valid inputs and properly reject invalid inputs with clear error messages
- No direct TOML parsing or environment variable access occurs outside of UnifiedConfigLoader in production code
- Code review confirms all new configuration loading code uses the unified approach

## Enforcement

- Verified by: Automated CI pipeline checks for UnifiedConfigLoader usage patterns
- Verified by: ESLint rules preventing direct configuration parsing imports
- Verified by: Code review checklist includes verification of configuration loading approach
- Verified by: Integration tests validate configuration loading behavior across all components
- Violation handling: CI pipeline fails if direct configuration parsing is detected outside approved exceptions
- Violation handling: Code review blocks merge requests that bypass UnifiedConfigLoader without documented exception
- Violation handling: Security scanning tools flag unvalidated configuration inputs
- Violation handling: Quarterly architecture reviews audit configuration loading patterns for compliance
- Exception process: Submit exception request to architecture team with justification and impact analysis
- Exception process: Provide evidence that UnifiedConfigLoader cannot meet the specific requirement
- Exception process: Document alternative validation and security measures
- Exception process: Include migration plan if exception is temporary
- Exception process: Obtain written approval from architecture team lead before implementation