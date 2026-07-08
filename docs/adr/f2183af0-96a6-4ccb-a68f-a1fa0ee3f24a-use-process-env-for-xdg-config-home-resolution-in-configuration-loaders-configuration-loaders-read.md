# Use process.env for XDG_CONFIG_HOME Resolution in Configuration Loaders: Configuration Loaders Read

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all configuration loading operations that resolve user-specific configuration paths.

## Context

- Configuration loaders in src/core/UnifiedConfigLoader.ts and src/core/ConfigLoader.ts require resolution of user-specific configuration directories following XDG Base Directory specification
- The system exposes public configuration loading contracts (UnifiedLoadOptions, loadUnifiedConfig, IAgentConfig, LoadedConfig, ConfigOptions) that must consistently locate configuration files across different user environments
- Environment variables provide the standard mechanism for runtime configuration path discovery without hardcoding user-specific paths
- Both configuration loaders access process.env.XDG_CONFIG_HOME to determine the base configuration directory, establishing a consistent pattern across the configuration subsystem

## Problem Statement

Configuration loaders must resolve user-specific configuration directories in a portable, environment-aware manner without embedding absolute paths or making assumptions about user home directory structure. The XDG Base Directory specification provides a standard, but its implementation requires accessing environment variables at runtime, which introduces a dependency on process.env and creates a boundary between configuration logic and runtime environment state.

## Decision

1. MUST: Configuration loaders MUST read XDG_CONFIG_HOME from process.env to resolve user-specific configuration directories

## Policy Block

- MUST Configuration loaders MUST read XDG_CONFIG_HOME from process.env to resolve user-specific configuration directories

In scope:
- All configuration loading operations in src/core/UnifiedConfigLoader.ts
- All configuration loading operations in src/core/ConfigLoader.ts
- Public API contracts: UnifiedLoadOptions, loadUnifiedConfig, IAgentConfig, LoadedConfig, ConfigOptions
- Runtime configuration path resolution for TOML and JSON configuration files

Out of scope:
- Configuration file parsing logic (TOML, JSON) after file location is resolved
- Configuration validation using zod schemas
- Configuration merging strategies
- MCP server configuration handling
- Agent-specific configuration beyond path resolution

Exceptions:
- EXC-001: Testing environments that require deterministic configuration paths
- EXC-002: Containerized or sandboxed environments with pre-configured paths

## Rationale

- Evidence shows both UnifiedConfigLoader.ts and ConfigLoader.ts access process.env.XDG_CONFIG_HOME, establishing a consistent pattern with 89.20% confidence across 2 files
- The XDG Base Directory specification is a widely-adopted standard for user-specific configuration on Unix-like systems, making environment variable access the idiomatic approach
- Runtime environment variable access enables portable configuration loading across different user accounts, deployment environments, and testing scenarios without code changes
- The pattern separates configuration location concerns (environment-dependent) from configuration parsing and validation concerns (environment-independent), supporting modularity

## Consequences

Positive:
- Configuration loaders work consistently across different user environments without hardcoded paths
- Testing can inject custom configuration paths through environment variable mocking
- Compliance with XDG Base Directory specification improves interoperability with other XDG-aware tools
- Clear separation between environment-dependent path resolution and environment-independent configuration processing

Negative:
- Configuration loading behavior depends on runtime environment state, making it harder to reason about in isolation
- Process.env access creates a global dependency that complicates pure functional testing approaches
- Missing or misconfigured XDG_CONFIG_HOME environment variable can cause configuration loading failures that are not apparent until runtime
- Environment variable access introduces a potential security boundary where malicious environment manipulation could redirect configuration loading

## Alternatives

- Pass configuration directory paths explicitly through function parameters (rejected)
  Rejected because: Requires all callers to know and provide configuration paths, violating encapsulation and making the API more complex. Does not support the XDG specification's intent of environment-driven configuration.
  When valid: Valid for library code that should not assume any environment conventions and must be explicitly configured by the host application
- Use a configuration registry or service locator pattern to abstract environment access (rejected)
  Rejected because: Adds architectural complexity and indirection for a straightforward environment variable read. The current pattern is idiomatic for Node.js applications and well-understood by developers.
  When valid: Valid for applications with complex environment abstraction requirements or multi-runtime support (Node.js, Deno, Bun) where process.env semantics differ
- Read XDG_CONFIG_HOME once at module initialization and cache the value (rejected)
  Rejected because: Prevents runtime environment changes from taking effect and complicates testing scenarios where environment variables need to change between test cases. Current evidence shows runtime access pattern.
  When valid: Valid for long-running services where configuration paths are guaranteed immutable after startup and startup performance is critical

## Risks

- Environment variable injection attacks could redirect configuration loading to attacker-controlled directories
  Mitigation: Validate resolved configuration paths to ensure they are within expected boundaries. Implement path traversal checks. Consider read-only configuration directory permissions.
  Owner: Security team and configuration subsystem maintainers
- Inconsistent environment setup across deployment environments could cause configuration loading failures that are difficult to diagnose
  Mitigation: Document XDG_CONFIG_HOME requirements clearly. Implement fallback behavior with clear logging. Add startup validation that checks environment variable presence and path accessibility.
  Owner: DevOps team and configuration subsystem maintainers
- Testing complexity increases due to global state dependency on process.env
  Mitigation: Provide test utilities for environment variable mocking. Document testing patterns in developer guides. Consider dependency injection for testability in future refactoring.
  Owner: Engineering team

## Implementation Notes

- Access process.env.XDG_CONFIG_HOME directly in configuration path resolution functions, following the pattern established in UnifiedConfigLoader.ts and ConfigLoader.ts
- Implement fallback logic when XDG_CONFIG_HOME is undefined, typically defaulting to $HOME/.config per XDG specification
- Log the resolved configuration directory path at debug level to aid troubleshooting in production environments
- Validate resolved paths before attempting file system operations to fail fast on misconfiguration
- Document the XDG_CONFIG_HOME dependency in public API documentation for loadUnifiedConfig and loadConfig functions

## Continuation Context


Verify commands:
- grep -r 'process\.env\.XDG_CONFIG_HOME' src/core/UnifiedConfigLoader.ts src/core/ConfigLoader.ts
- grep -r 'XDG_CONFIG_HOME' src/core/*.ts | grep -v 'process.env' | wc -l | grep '^0$'
- npm test -- --grep 'configuration.*XDG'

Accept when:
- grep confirms process.env.XDG_CONFIG_HOME is accessed in both UnifiedConfigLoader.ts and ConfigLoader.ts
- No hardcoded XDG_CONFIG_HOME values exist in configuration loader files (grep returns 0 matches)
- Configuration loading tests pass with mocked XDG_CONFIG_HOME environment variables

## Enforcement

- Verified by: Code review checklist includes verification of process.env.XDG_CONFIG_HOME usage in configuration loading code
- Verified by: Static analysis grep patterns in CI pipeline detect hardcoded configuration paths
- Verified by: Integration tests validate configuration loading with various XDG_CONFIG_HOME values
- Violation handling: CI pipeline fails if hardcoded configuration paths are detected in configuration loader modules
- Violation handling: Code review blocks merge if configuration loading bypasses process.env.XDG_CONFIG_HOME
- Violation handling: Runtime warnings logged if configuration loading falls back due to missing XDG_CONFIG_HOME
- Exception process: Exception requests must document why process.env access is not suitable for the specific use case
- Exception process: Architecture review required for any alternative configuration path resolution mechanism
- Exception process: Approved exceptions must be documented in code comments with reference to this ADR and the exception approval