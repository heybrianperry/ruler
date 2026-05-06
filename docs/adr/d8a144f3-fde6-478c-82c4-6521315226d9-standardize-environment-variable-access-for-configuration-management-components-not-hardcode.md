# Standardize Environment Variable Access for Configuration Management: Components Not Hardcode

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all runtime components that require configuration management, including agents, core services, MCP integrations, and VS Code extensions.

## Context

- The codebase contains multiple agent implementations (GeminiCliAgent, OpenCodeAgent, MistralVibeAgent, RooCodeAgent, etc.) and core services that require runtime configuration
- Configuration values such as API keys, endpoints, feature flags, and operational parameters must be externalized from code to support different deployment environments
- A consistent pattern for accessing environment variables has emerged across 17 files with 91.14% confidence, indicating an established architectural practice
- The pattern spans agents, configuration loaders, MCP propagation, path resolution, and VS Code settings, demonstrating system-wide adoption
- Environment-based configuration enables separation of concerns between code and deployment-specific settings, supporting security and operational flexibility

## Problem Statement

Without a standardized approach to environment variable access, configuration management becomes inconsistent across components, leading to potential security vulnerabilities (hardcoded secrets), deployment difficulties, testing challenges, and maintenance overhead. The system needs a unified pattern for externalizing configuration that works reliably across all runtime contexts.

## Decision

1. MUST_NOT: Components MUST NOT hardcode sensitive configuration values (API keys, tokens, passwords) directly in source code

## Policy Block

- MUST_NOT Components MUST NOT hardcode sensitive configuration values (API keys, tokens, passwords) directly in source code

In scope:
- All agent implementations (CLI agents, code agents, specialized agents)
- Core configuration loaders (ConfigLoader, UnifiedConfigLoader)
- MCP integration components and propagation logic
- Path resolution and filesystem configuration
- VS Code extension settings and runtime configuration
- Apply engine and core service initialization

Out of scope:
- Build-time constants that never vary by environment
- Type definitions and interface declarations
- Static utility functions that don't require configuration
- Test fixtures and mock data (which may use alternative configuration approaches)

Exceptions:
- EXC-001: Development and testing environments where hardcoded defaults improve developer experience
- EXC-002: Configuration values are derived from other environment variables or computed at runtime

## Rationale

- Pattern detected across 17 files with 91.14% confidence indicates this is an established and proven architectural practice in the codebase
- Environment variable-based configuration is an industry standard practice (12-factor app methodology) that enables secure, flexible deployment across different environments
- The pattern's presence in critical components (agents, configuration loaders, MCP integration) demonstrates its importance to system architecture and operational requirements
- Consistent configuration access patterns reduce cognitive load for developers and minimize the risk of configuration-related bugs or security vulnerabilities

## Consequences

Positive:
- Enhanced security through externalization of sensitive configuration values from source code
- Improved deployment flexibility allowing the same codebase to run in development, staging, and production with different configurations
- Better testability through ability to inject test-specific configuration via environment variables
- Simplified configuration management with a single, well-understood mechanism across all components
- Reduced risk of accidentally committing secrets to version control

Negative:
- Increased operational complexity in managing environment variables across different deployment contexts
- Potential runtime failures if required environment variables are not properly set, requiring robust validation
- Debugging challenges when configuration issues arise, as values are not visible in code
- Documentation burden to maintain accurate lists of required and optional environment variables for each component

## Alternatives

- Use configuration files (JSON, YAML, TOML) for all configuration management (rejected)
  Rejected because: Configuration files are less secure for sensitive values, harder to manage in containerized environments, and don't align with the detected pattern showing strong preference for environment variables
  When valid: May be used as a supplementary configuration source for complex, structured configuration that complements environment variables
- Implement a centralized configuration service or secrets management system (deferred)
  Rejected because: Adds significant infrastructure complexity and dependencies; current environment variable approach is working well as evidenced by widespread adoption
  When valid: Should be reconsidered when the system scales to require centralized secret rotation, audit logging, or multi-tenant configuration isolation
- Use command-line arguments for all runtime configuration (rejected)
  Rejected because: Command-line arguments are visible in process listings (security risk), difficult to manage for large numbers of configuration values, and don't integrate well with container orchestration
  When valid: Appropriate for user-facing CLI options that override environment-based defaults

## Risks

- Missing or misconfigured environment variables cause runtime failures in production
  Mitigation: Implement comprehensive validation at startup with clear error messages; use infrastructure-as-code to manage environment configuration; implement health checks that verify configuration
  Owner: Engineering team and DevOps
- Environment variables containing sensitive data may be logged or exposed through error messages
  Mitigation: Implement sanitization in logging frameworks; never log raw environment variable values; use structured logging with explicit field control
  Owner: Engineering team
- Inconsistent environment variable naming across components leads to confusion and misconfiguration
  Mitigation: Establish and document naming conventions; implement automated checks for naming compliance; maintain a central registry of all environment variables
  Owner: Architecture team

## Implementation Notes

- Use TypeScript's type system to define configuration interfaces and validate environment variable access at compile time where possible
- Implement a centralized configuration loader that handles environment variable access, validation, type conversion, and default values
- Consider using a library like 'dotenv' for local development to load environment variables from .env files (never commit .env files to version control)
- Document all environment variables in a central location (e.g., README.md or docs/configuration.md) with descriptions, types, required/optional status, and default values
- Implement configuration validation that runs at application startup and fails fast with actionable error messages for missing or invalid values

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' src/ --include='*.ts' | wc -l
- grep -r 'API_KEY.*=.*['\"].*['\"]' src/ --include='*.ts' | grep -v 'process.env' || echo 'No hardcoded API keys found'
- find src/ -name '*ConfigLoader*.ts' -o -name '*config*.ts' | xargs grep -l 'process.env'

Accept when:
- All agent implementations and configuration loaders access runtime configuration via process.env
- No hardcoded sensitive values (API keys, tokens, passwords) are found in source code
- Configuration loaders implement validation and provide clear error messages for missing required environment variables
- Environment variables are documented with their purpose, type, and whether they are required or optional

## Enforcement

- Verified by: Automated code review checks scanning for hardcoded secrets and configuration values
- Verified by: CI pipeline validation ensuring configuration loaders properly handle missing environment variables
- Verified by: Security scanning tools checking for exposed credentials in source code
- Verified by: Manual code review for new agent implementations and configuration components
- Violation handling: CI pipeline fails if hardcoded secrets or API keys are detected in source code
- Violation handling: Code review blocks merge if configuration access doesn't follow established patterns
- Violation handling: Security team notification for any detected credential exposure
- Violation handling: Immediate remediation required for any hardcoded sensitive values discovered in production code
- Exception process: Developer submits exception request to tech lead with justification and risk assessment
- Exception process: Tech lead reviews against policy exceptions (EXC-001, EXC-002) and approves or denies
- Exception process: Approved exceptions must be documented in code comments and tracked in architecture decision log
- Exception process: Exceptions are reviewed quarterly and must be re-justified or remediated