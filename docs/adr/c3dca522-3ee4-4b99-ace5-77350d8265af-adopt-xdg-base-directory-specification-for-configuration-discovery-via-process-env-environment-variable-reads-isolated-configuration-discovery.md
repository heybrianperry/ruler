# Adopt XDG Base Directory Specification for Configuration Discovery via process.env: Environment Variable Reads Isolated Configuration Discovery

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- CLI applications require a standardized mechanism to discover user configuration files across different Unix-like environments without hardcoding filesystem paths
- The XDG Base Directory specification provides a cross-platform standard for organizing user configuration, cache, and data files on Unix-like systems through environment variables
- Configuration file location must be discoverable at CLI handler initialization time before processing user commands
- The project uses Node.js runtime environment variables as the primary configuration source mechanism

## Problem Statement

CLI handlers need to locate user configuration files in a platform-appropriate manner that respects user preferences and system conventions. Hardcoded paths reduce portability and prevent users from organizing their configuration files according to their environment's standards. A standardized discovery mechanism is required that integrates with the Node.js runtime environment.

## Decision

1. SHOULD: Environment variable reads SHOULD be isolated to configuration discovery modules to maintain separation of concerns

## Policy Block

- SHOULD Environment variable reads SHOULD be isolated to configuration discovery modules to maintain separation of concerns

In scope:
- CLI handler modules that require user configuration file access
- Configuration discovery and initialization code paths
- Unix-like operating systems where XDG Base Directory specification is applicable

Out of scope:
- Windows-specific configuration discovery (requires alternative platform conventions)
- System-wide configuration files (typically use /etc or platform-specific locations)
- Temporary or cache directories (covered by separate XDG variables)
- Runtime configuration passed via command-line arguments or stdin

## Rationale

- The XDG Base Directory specification is a widely adopted standard on Unix-like systems, providing predictable configuration file locations that users and system administrators expect
- Reading process.env.XDG_CONFIG_HOME allows users to customize configuration directory location without code changes, supporting diverse deployment environments and user preferences
- Evidence shows this pattern implemented in CLI handlers (src/cli/handlers.ts) with significance 0.89, indicating intentional architectural choice for configuration source hierarchy
- Node.js process.env API provides direct, synchronous access to environment variables at initialization time, suitable for configuration discovery before command execution

## Consequences

Positive:
- Configuration files follow platform conventions, improving user experience and system integration on Unix-like environments
- Users can relocate configuration directories by setting XDG_CONFIG_HOME without modifying application code
- Compliance with XDG specification enables interoperability with other XDG-aware tools and system management utilities
- Environment variable-based discovery supports containerized and multi-tenant deployments where filesystem paths vary

Negative:
- Platform-specific behavior creates divergence between Unix-like and Windows environments, requiring separate configuration discovery logic
- Environment variable dependency introduces runtime configuration that may not be obvious to developers unfamiliar with XDG conventions
- Testing requires environment variable mocking or injection, adding complexity to test setup
- Fallback logic must be maintained for environments where XDG_CONFIG_HOME is unset, increasing code paths

## Alternatives

- Hardcode configuration directory to a fixed path relative to user home directory (rejected)
  Rejected because: Violates XDG Base Directory specification, prevents user customization, and reduces portability across different Unix-like environments with varying conventions
  When valid: Acceptable for Windows-only applications or internal tools with controlled deployment environments
- Use a third-party configuration management library that abstracts platform differences (rejected)
  Rejected because: Adds external dependency for functionality achievable with Node.js built-in APIs; evidence shows direct process.env usage pattern already adopted
  When valid: Justified when cross-platform configuration discovery requires complex logic beyond XDG specification compliance
- Require configuration file path as mandatory command-line argument (rejected)
  Rejected because: Degrades user experience by forcing explicit path specification on every invocation; prevents convention-based discovery
  When valid: Appropriate for one-off scripts or when configuration location must be explicitly controlled for security or compliance reasons

## Risks

- Environment variable may be undefined or point to non-existent directory, causing configuration discovery failure
  Mitigation: Implement fallback logic to default XDG-compliant path ($HOME/.config) when XDG_CONFIG_HOME is unset; validate directory existence before use
  Owner: engineering team
- Cross-platform inconsistency if Windows or non-Unix environments attempt to use XDG-specific discovery logic
  Mitigation: Implement platform detection and provide Windows-specific configuration discovery path; document platform-specific behavior in user-facing documentation
  Owner: engineering team
- Security implications if XDG_CONFIG_HOME points to untrusted or world-writable directory
  Mitigation: Validate directory permissions before reading configuration files; document security expectations for configuration directory ownership
  Owner: engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Fallback logic should follow XDG specification defaults: when XDG_CONFIG_HOME is unset, default to $HOME/.config as specified in the XDG Base Directory specification version 0.8
- Configuration path construction should use the runtime's path module to ensure proper path separator handling and normalization across different Unix-like systems
- Consider caching the resolved configuration directory path after first access to avoid repeated environment variable reads and path resolution operations

## Continuation Context


Verify commands:
- Locate the project's test suite and identify tests covering configuration discovery; execute tests that verify XDG_CONFIG_HOME environment variable handling
- Inspect the CLI handler module to confirm process.env.XDG_CONFIG_HOME access pattern and fallback logic implementation
- Set XDG_CONFIG_HOME to a test directory and verify the CLI locates configuration files in the specified location; unset the variable and verify fallback behavior

Accept when:
- Configuration discovery code reads XDG_CONFIG_HOME from process.env and uses the value as the base configuration directory
- Fallback logic provides XDG-compliant default path when XDG_CONFIG_HOME is undefined
- Tests verify configuration discovery behavior with both set and unset XDG_CONFIG_HOME environment variable states

## Enforcement

- Verified by: Code review verification that configuration discovery uses process.env.XDG_CONFIG_HOME
- Verified by: Unit tests covering XDG_CONFIG_HOME presence and absence scenarios
- Verified by: Integration tests validating configuration file discovery in XDG-compliant directory structure
- Violation handling: Code review rejection for configuration discovery code that bypasses XDG_CONFIG_HOME
- Violation handling: Test failures when configuration discovery does not respect environment variable settings
- Violation handling: Documentation updates required when platform-specific exceptions are introduced
- Exception process: Platform-specific configuration discovery (Windows, macOS) may use alternative environment variables or conventions with architectural review approval
- Exception process: System-wide configuration files may use fixed paths when XDG specification explicitly excludes them from user configuration scope
- Exception process: Exceptions must be documented in code comments with rationale and approved by configuration architecture owner