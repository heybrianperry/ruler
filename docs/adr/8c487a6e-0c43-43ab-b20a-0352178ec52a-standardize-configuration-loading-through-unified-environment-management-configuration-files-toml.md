# Standardize Configuration Loading Through Unified Environment Management: Configuration Files Toml

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all configuration loading operations across agents and core engine components.

## Context

- The codebase contains multiple agent implementations (AmazonQCliAgent, FirebenderAgent, OpenCodeAgent, RooCodeAgent, CrushAgent, QwenCodeAgent, MistralVibeAgent) that require consistent configuration loading mechanisms
- A UnifiedConfigLoader component exists alongside an apply-engine that processes configuration, indicating a centralized approach to environment and configuration management
- Test files (test_manual_toml.js, test_toml_options.js) demonstrate TOML-based configuration testing, suggesting TOML as a primary configuration format
- The pattern appears across 11 files with 91.03% confidence, indicating a well-established architectural practice
- Multiple agents sharing similar configuration patterns suggests a need for standardization to prevent configuration drift and ensure consistent behavior

## Problem Statement

Without a unified configuration loading strategy, agent implementations risk inconsistent environment setup, duplicated configuration logic, and fragile runtime behavior. The system needs a standardized approach to load, validate, and apply configuration across diverse agent types while maintaining flexibility for agent-specific requirements.

## Decision

1. MUST: Configuration files MUST be in TOML format for consistency and human readability

## Policy Block

- MUST Configuration files MUST be in TOML format for consistency and human readability

In scope:
- All agent implementations (AmazonQCliAgent, FirebenderAgent, OpenCodeAgent, RooCodeAgent, CrushAgent, QwenCodeAgent, MistralVibeAgent, and future agents)
- Core engine components that require runtime configuration
- Test suites that validate configuration loading behavior
- TOML configuration files used for environment setup

Out of scope:
- Hardcoded default values within agent constructors (allowed as fallbacks)
- Runtime parameter overrides passed programmatically
- Temporary configuration used in isolated unit tests
- Third-party library configuration that doesn't integrate with the unified system

Exceptions:
- EXC-001: An agent requires real-time configuration updates that cannot be handled through file-based TOML loading
- EXC-002: Legacy agents being migrated incrementally to the unified configuration system

## Rationale

- Pattern detected across 11 files with 91.03% significance indicates this is an established and successful architectural practice
- Centralized configuration loading through UnifiedConfigLoader reduces code duplication and ensures consistent behavior across all agents
- TOML format provides human-readable configuration with strong typing support, reducing configuration errors
- The apply-engine pattern separates configuration loading from application, enabling better testing and validation

## Consequences

Positive:
- Consistent configuration behavior across all agent implementations reduces debugging time and operational complexity
- Centralized validation logic catches configuration errors early in the startup process
- TOML format is human-readable and version-control friendly, improving configuration management
- New agents can be onboarded quickly by reusing the established configuration infrastructure

Negative:
- Adds a dependency on the UnifiedConfigLoader component, creating a potential single point of failure
- TOML format may have limitations for complex nested configurations compared to JSON or YAML
- Agent developers must learn the unified configuration schema rather than implementing custom solutions
- Migration effort required for any existing agents using alternative configuration approaches

## Alternatives

- Allow each agent to implement its own configuration loading mechanism (rejected)
  Rejected because: Leads to inconsistent behavior, duplicated code, and increased maintenance burden across multiple agent implementations
  When valid: Only valid for prototype or experimental agents not intended for production use
- Use JSON instead of TOML for configuration files (rejected)
  Rejected because: JSON lacks native support for comments and is less human-readable for configuration purposes; TOML is already established in the codebase
  When valid: Could be reconsidered if complex nested structures become a significant limitation
- Use environment variables exclusively without configuration files (rejected)
  Rejected because: Environment variables are difficult to manage for complex configurations and don't support structured data well
  When valid: Appropriate for deployment-specific overrides or sensitive credentials, but not as the primary configuration mechanism

## Risks

- UnifiedConfigLoader becomes a bottleneck or single point of failure if not properly maintained
  Mitigation: Implement comprehensive test coverage for UnifiedConfigLoader; establish clear ownership and maintenance responsibilities; consider circuit breaker patterns for graceful degradation
  Owner: Core Platform Team
- TOML format limitations may constrain future configuration requirements
  Mitigation: Monitor configuration complexity over time; maintain abstraction layer that could support multiple formats if needed; document known TOML limitations
  Owner: Architecture Team
- Configuration schema changes could break existing agents without proper versioning
  Mitigation: Implement configuration schema versioning; maintain backward compatibility; use validation to detect schema mismatches early
  Owner: Engineering Team

## Implementation Notes

- New agents should import UnifiedConfigLoader and call it during initialization before any configuration-dependent operations
- Configuration files should be placed in a standard location (e.g., config/ directory) with clear naming conventions
- Use the apply-engine to transform loaded configuration into runtime state, keeping loading and application concerns separated
- Add validation rules to catch common configuration errors (missing required fields, invalid values, type mismatches) at load time
- Document the configuration schema for each agent in its README or documentation, including required and optional fields

## Continuation Context


Verify commands:
- grep -r 'UnifiedConfigLoader' src/agents/ | wc -l
- find . -name '*.toml' -path '*/config/*' | wc -l
- grep -r 'new.*ConfigLoader' src/agents/ | grep -v 'UnifiedConfigLoader' && echo 'Custom loaders found' || echo 'Compliant'

Accept when:
- All agent files in src/agents/ import and use UnifiedConfigLoader for configuration loading
- Configuration files are in TOML format and located in standard config directories
- No custom configuration parsers are implemented that bypass the unified loading mechanism
- Test files validate configuration loading through the standard UnifiedConfigLoader interface

## Enforcement

- Verified by: Automated CI checks scanning for UnifiedConfigLoader usage in all agent implementations
- Verified by: Code review checklist requiring unified configuration approach for new agents
- Verified by: Static analysis tools detecting custom configuration parser implementations
- Verified by: Integration tests validating configuration loading behavior across all agents
- Violation handling: CI pipeline fails if agents don't use UnifiedConfigLoader
- Violation handling: Code review blocks merge requests that introduce custom configuration loaders without approved exceptions
- Violation handling: Architecture review required for any deviation from the standard configuration approach
- Violation handling: Quarterly audits to identify and remediate configuration loading inconsistencies
- Exception process: Submit exception request to architecture team with detailed justification
- Exception process: Document technical constraints that prevent use of UnifiedConfigLoader
- Exception process: Propose alternative approach with equivalent safety and consistency guarantees
- Exception process: Obtain written approval from architecture team lead before implementation
- Exception process: Add exception to tracking system with review date for re-evaluation