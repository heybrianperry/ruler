# Use XDG_CONFIG_HOME Environment Variable for Global Configuration Discovery: Error Messages Related

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The system requires a mechanism to locate global configuration directories across different operating systems and user environments
- XDG Base Directory specification provides a standardized approach for configuration file placement on Unix-like systems
- The codebase accesses process.env.XDG_CONFIG_HOME in both src/core/FileSystemUtils.ts and src/cli/handlers.ts, indicating a cross-module configuration discovery pattern
- Error handling around global config directory access suggests the system must gracefully handle missing or inaccessible configuration paths

## Problem Statement

The system needs a consistent, cross-platform method to discover global configuration directories without hardcoding paths, while respecting user-defined configuration locations and operating system conventions.

## Decision

1. SHOULD: Error messages related to configuration directory access SHOULD be logged using console.error with descriptive context including the attempted directory path

## Policy Block

- SHOULD Error messages related to configuration directory access SHOULD be logged using console.error with descriptive context including the attempted directory path

In scope:
- Global configuration directory discovery in src/core/FileSystemUtils.ts
- CLI handler configuration resolution in src/cli/handlers.ts
- Any module that needs to locate user or system-wide configuration files

Out of scope:
- Project-local configuration files within .ruler directories
- Runtime configuration passed via command-line arguments
- Hardcoded default values that do not involve filesystem discovery

## Rationale

- The XDG Base Directory specification is a widely adopted standard on Unix-like systems, providing predictable configuration file locations that respect user preferences
- Reading from process.env.XDG_CONFIG_HOME enables users to override default configuration paths without code changes, supporting flexible deployment scenarios
- The pattern appears in 2 distinct files (FileSystemUtils and handlers), indicating this is an established architectural convention rather than isolated usage
- Error handling around configuration directory access ensures the system degrades gracefully when configuration sources are unavailable

## Consequences

Positive:
- Enables portable configuration discovery across different Unix-like operating systems and user environments
- Respects user-defined configuration preferences through standard environment variables
- Reduces coupling to hardcoded filesystem paths, improving testability and deployment flexibility
- Provides consistent configuration discovery behavior across multiple modules through shared utilities

Negative:
- Introduces dependency on external environment state (process.env), which may complicate testing and debugging
- Requires additional error handling logic to manage cases where XDG_CONFIG_HOME is unset or points to invalid locations
- May create confusion on non-Unix systems where XDG conventions are not standard
- Adds runtime overhead for environment variable lookups during configuration discovery

## Alternatives

- Hardcode configuration paths to standard locations like /etc or ~/.config (rejected)
  Rejected because: Hardcoded paths prevent users from customizing configuration locations and reduce portability across different system configurations
  When valid: Only appropriate for systems with guaranteed fixed filesystem layouts and no user customization requirements
- Accept configuration directory paths exclusively via command-line arguments (rejected)
  Rejected because: Requiring explicit path arguments for every invocation creates poor user experience and prevents system-wide configuration defaults
  When valid: Suitable for one-off operations or scripts where explicit control is more important than convenience
- Use a configuration library that abstracts platform-specific config paths (deferred)
  Rejected because: Adds external dependency overhead; current implementation with XDG_CONFIG_HOME provides sufficient functionality for observed use cases
  When valid: If cross-platform support expands to Windows or macOS with platform-specific conventions, a library like 'env-paths' may be warranted

## Risks

- XDG_CONFIG_HOME may be unset or point to a non-existent directory, causing configuration discovery to fail
  Mitigation: Implement fallback logic to default locations (e.g., ~/.config) and comprehensive error handling with clear user-facing messages
  Owner: engineering team
- Environment variable injection or manipulation could redirect configuration discovery to malicious locations
  Mitigation: Validate resolved paths using security utilities (e.g., assertContainingDirectoryInsideRoot, assertNotSymbolicLink) as evidenced in FileSystemUtils
  Owner: security team
- Inconsistent environment variable handling across different execution contexts (CLI, tests, CI) may lead to configuration discovery failures
  Mitigation: Document required environment setup and provide test fixtures that mock process.env consistently
  Owner: engineering team

## Implementation Notes

- Import configuration discovery utilities from src/core/FileSystemUtils.ts to ensure consistent behavior across modules
- Always wrap XDG_CONFIG_HOME access with try-catch blocks and log errors using console.error with descriptive context
- Implement fallback logic that checks XDG_CONFIG_HOME first, then falls back to platform-specific defaults like ~/.config on Unix systems
- Use path resolution utilities (path.resolve, path.join) to normalize paths obtained from environment variables before filesystem operations

## Continuation Context


Verify commands:
- grep -r 'process\.env\.XDG_CONFIG_HOME' src/ --include='*.ts'
- grep -r 'console\.error.*global.*config' src/ --include='*.ts'
- grep -r 'assertNotSymbolicLink\|assertContainingDirectoryInsideRoot' src/core/FileSystemUtils.ts

Accept when:
- All modules requiring global configuration discovery read XDG_CONFIG_HOME from process.env
- Error handling is present for all configuration directory access operations with descriptive logging
- Security validation utilities are applied to paths derived from environment variables

## Enforcement

- Verified by: Code review checking for direct environment variable access patterns
- Verified by: Static analysis scanning for process.env.XDG_CONFIG_HOME usage
- Verified by: Integration tests validating configuration discovery with various XDG_CONFIG_HOME values
- Violation handling: Pull requests introducing hardcoded configuration paths are rejected during code review
- Violation handling: Linting rules flag direct filesystem path construction without environment variable checks
- Violation handling: CI pipeline fails if configuration discovery tests do not cover XDG_CONFIG_HOME scenarios
- Exception process: Document the specific use case requiring an exception in the module's README or inline comments
- Exception process: Obtain approval from the architecture review board for deviations from XDG-based discovery
- Exception process: Add a tracking issue to revisit the exception and align with the standard pattern in future refactoring