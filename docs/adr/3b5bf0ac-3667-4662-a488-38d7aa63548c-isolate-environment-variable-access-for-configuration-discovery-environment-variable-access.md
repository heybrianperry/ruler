# Isolate Environment Variable Access for Configuration Discovery: Environment Variable Access

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase accesses process.env.XDG_CONFIG_HOME directly in multiple modules (src/core/FileSystemUtils.ts and src/cli/handlers.ts) to resolve user-specific configuration directories
- Environment variables represent runtime configuration sources that cross trust boundaries between the operating system, deployment environment, and application code
- The XDG Base Directory specification defines standard locations for user configuration files on Unix-like systems, requiring environment variable inspection at runtime
- Direct process.env access couples configuration discovery logic to Node.js runtime primitives and creates implicit dependencies on environment state

## Problem Statement

Direct access to process.env for configuration discovery scatters environment variable handling across multiple modules, making it difficult to audit, test, and secure configuration sources. This pattern creates implicit runtime dependencies and complicates efforts to validate, sanitize, or mock environment state during testing and development.

## Decision

1. SHOULD: Environment variable access SHOULD be abstracted behind interfaces to enable testing with mock configuration sources

## Policy Block

- SHOULD Environment variable access SHOULD be abstracted behind interfaces to enable testing with mock configuration sources

In scope:
- All modules that read environment variables for application configuration
- Configuration discovery logic that resolves user-specific or system-wide settings
- Runtime initialization code that depends on environment state

Out of scope:
- Third-party libraries that manage their own environment variable access
- Test fixtures that explicitly set process.env for test scenarios
- Build-time or compile-time environment variable substitution

## Rationale

- The IR evidence shows process.env.XDG_CONFIG_HOME accessed in two separate modules (FileSystemUtils.ts and handlers.ts), indicating a pattern of distributed environment variable handling
- Centralizing environment variable access improves auditability for security reviews, particularly when configuration sources cross trust boundaries
- Isolated configuration modules enable consistent validation, type checking, and default value handling across the codebase
- Abstraction of environment access facilitates testing by allowing mock configuration injection without modifying global process.env state

## Consequences

Positive:
- Improved testability through dependency injection of configuration sources
- Centralized documentation of all environment variables used by the application
- Easier security auditing of configuration sources and potential injection vectors
- Consistent validation and error handling for missing or malformed environment variables

Negative:
- Additional abstraction layer adds indirection between business logic and configuration sources
- Requires refactoring existing direct process.env access patterns across multiple modules
- May introduce performance overhead if configuration abstraction is not efficiently implemented
- Increases initial development complexity for simple configuration reads

## Alternatives

- Continue direct process.env access throughout the codebase (rejected)
  Rejected because: Scattered environment variable access makes security auditing difficult and creates implicit dependencies that complicate testing and maintenance
  When valid: In small scripts or prototypes where configuration complexity does not justify abstraction overhead
- Use a third-party configuration management library (e.g., dotenv, config, convict) (deferred)
  Rejected because: Not rejected; deferred pending evaluation of library compatibility with existing architecture
  When valid: When configuration requirements grow to include multiple sources (files, environment, CLI args) and validation schemas
- Implement a singleton configuration service with lazy initialization (accepted)
  When valid: When configuration values are read frequently and caching improves performance without introducing stale data issues

## Risks

- Incomplete migration leaves some modules using direct process.env access while others use the abstraction layer
  Mitigation: Implement linting rules to detect direct process.env access and require explicit exceptions
  Owner: engineering team
- Configuration abstraction introduces caching bugs where environment changes are not reflected at runtime
  Mitigation: Document cache invalidation behavior and provide explicit refresh mechanisms for dynamic configuration scenarios
  Owner: engineering team
- Over-abstraction creates unnecessary complexity for simple environment variable reads
  Mitigation: Provide lightweight accessor functions for simple cases while reserving complex validation for critical configuration
  Owner: engineering team

## Implementation Notes

- Create a dedicated configuration module (e.g., src/core/config.ts) that exports typed accessors for all environment variables
- Document each environment variable with its purpose, expected format, default value, and security implications
- Refactor existing process.env.XDG_CONFIG_HOME access in FileSystemUtils.ts and handlers.ts to use the centralized configuration module
- Add unit tests that verify configuration behavior with various environment states using mock configuration sources

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' src/ --exclude-dir=node_modules | grep -v 'src/core/config.ts' | wc -l
- npm test -- --testPathPattern=config.test.ts
- eslint src/ --rule 'no-process-env: error' --ignore-pattern 'src/core/config.ts'

Accept when:
- All direct process.env access is isolated to designated configuration modules or has documented exceptions
- Configuration module includes type-safe accessors with validation for all environment variables
- Unit tests demonstrate configuration behavior can be tested without modifying global process.env

## Enforcement

- Verified by: ESLint rule no-process-env enforced in CI pipeline with explicit allowlist for configuration modules
- Verified by: Code review checklist includes verification that new environment variable access uses configuration abstraction
- Verified by: Automated tests verify configuration module behavior with various environment states
- Violation handling: CI pipeline fails on direct process.env access outside designated configuration modules
- Violation handling: Pull requests with violations are blocked until refactored or exception is documented
- Violation handling: Security review required for any new environment variable access patterns
- Exception process: Document exception rationale in code comments explaining why direct access is necessary
- Exception process: Add ESLint disable comment with ticket reference for tracking technical debt
- Exception process: Security team approval required for environment variables containing sensitive data