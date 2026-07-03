# Isolate Environment Variable Access for Configuration Discovery: Modules That Require

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase accesses process.env.XDG_CONFIG_HOME directly in multiple modules (src/core/FileSystemUtils.ts and src/cli/handlers.ts) to resolve configuration directory paths
- Configuration discovery logic is distributed across core utilities and CLI handlers, both requiring runtime environment variable access
- The XDG_CONFIG_HOME environment variable follows the XDG Base Directory specification for user-specific configuration file locations
- Direct process.env access creates implicit runtime dependencies and complicates testing, mocking, and configuration validation

## Problem Statement

Direct access to process.env.XDG_CONFIG_HOME in multiple modules creates tight coupling to the Node.js runtime environment, making configuration sources difficult to test, validate, or substitute. This pattern lacks centralized control over environment variable access and exposes secrets handling concerns across the public API surface.

## Decision

1. MUST: Modules that require configuration values MUST receive them through dependency injection or explicit parameters rather than direct process.env access

## Policy Block

- MUST Modules that require configuration values MUST receive them through dependency injection or explicit parameters rather than direct process.env access

In scope:
- All runtime configuration derived from environment variables
- XDG_CONFIG_HOME and other XDG Base Directory specification variables
- Configuration path resolution in core utilities and CLI handlers
- Any module that currently accesses process.env directly

Out of scope:
- Build-time environment variables used by bundlers or compilers
- Environment variables accessed by third-party libraries
- System environment variables read by Node.js runtime itself (NODE_ENV, NODE_PATH)
- Temporary debugging or development-only environment variable access

Exceptions:
- EXC-001: Legacy code during migration period before configuration module is fully implemented

## Rationale

- Evidence shows process.env.XDG_CONFIG_HOME accessed directly in both src/core/FileSystemUtils.ts and src/cli/handlers.ts, indicating distributed configuration responsibility
- Centralizing environment variable access improves testability by allowing configuration injection and mocking without modifying global process.env
- The security.secrets_handling facet detection indicates environment variables may contain sensitive configuration paths that require controlled access patterns
- Type-safe configuration accessors prevent runtime errors from missing or malformed environment variables

## Consequences

Positive:
- Improved testability through dependency injection and configuration mocking
- Centralized validation and default value handling for environment variables
- Reduced coupling between business logic and Node.js runtime environment
- Easier to audit and control access to potentially sensitive configuration sources

Negative:
- Requires refactoring existing direct process.env access across multiple modules
- Adds an additional abstraction layer that developers must understand
- May introduce slight performance overhead if configuration caching is not implemented
- Increases initial development effort to establish configuration module infrastructure

## Alternatives

- Continue with direct process.env access throughout the codebase (rejected)
  Rejected because: Maintains tight coupling to runtime environment, complicates testing, and distributes secrets handling concerns across multiple modules without centralized control
  When valid: Only acceptable for trivial scripts or prototypes with no testing requirements
- Use a third-party configuration library (e.g., dotenv, convict, config) (deferred)
  Rejected because: Adds external dependency and may be over-engineered for current needs, but could be reconsidered if configuration complexity grows significantly
  When valid: When configuration sources expand beyond environment variables to include files, remote sources, or complex validation schemas
- Pass process.env as a parameter to all functions requiring configuration (rejected)
  Rejected because: Creates verbose function signatures and still lacks type safety and validation; does not solve the secrets handling concern
  When valid: Never; dependency injection of parsed configuration objects is superior

## Risks

- Incomplete migration leaves some modules still accessing process.env directly, creating inconsistent patterns
  Mitigation: Use linting rules or grep-based CI checks to detect direct process.env access; maintain migration tracking document
  Owner: Engineering team
- Configuration module becomes a bottleneck or single point of failure if not designed for extensibility
  Mitigation: Design configuration module with clear interfaces, plugin support, and comprehensive error handling from the start
  Owner: Architecture team
- Developers may bypass configuration module for convenience, especially under time pressure
  Mitigation: Enforce through code review guidelines, automated linting, and clear documentation of the pattern
  Owner: Engineering team

## Implementation Notes

- Create a dedicated src/config module that exports typed configuration accessors (e.g., getXdgConfigHome())
- Refactor src/core/FileSystemUtils.ts and src/cli/handlers.ts to receive configuration through constructor injection or function parameters
- Implement configuration validation with clear error messages for missing or invalid environment variables
- Add unit tests for configuration module that mock process.env to verify default value handling and validation logic

## Continuation Context


Verify commands:
- grep -r 'process\.env\.XDG_CONFIG_HOME' src/ --exclude-dir=config --exclude-dir=__tests__
- grep -r 'process\.env\.' src/ | grep -v 'src/config/' | grep -v '__tests__' | grep -v 'NODE_ENV'
- npm test -- --testPathPattern=config --coverage

Accept when:
- No direct process.env.XDG_CONFIG_HOME access exists outside the dedicated configuration module
- All configuration consumers receive configuration through dependency injection or parameters
- Configuration module has >90% test coverage with mocked environment variable scenarios

## Enforcement

- Verified by: Automated grep-based checks in CI pipeline detecting process.env access patterns
- Verified by: ESLint rule prohibiting direct process.env access outside configuration module
- Verified by: Code review checklist requiring configuration injection verification
- Violation handling: CI build fails if direct process.env access detected outside allowed locations
- Violation handling: Pull requests blocked until configuration access is refactored to use configuration module
- Violation handling: Violations logged and reported in weekly architecture review meetings
- Exception process: Developer submits exception request with justification to tech lead
- Exception process: Tech lead evaluates whether exception is temporary (migration) or permanent (system variable)
- Exception process: Approved exceptions documented in code with TODO comments and tracked in migration backlog