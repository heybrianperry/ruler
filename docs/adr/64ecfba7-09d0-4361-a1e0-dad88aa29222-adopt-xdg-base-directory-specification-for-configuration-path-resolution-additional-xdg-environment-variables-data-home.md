# Adopt XDG Base Directory Specification for Configuration Path Resolution: Additional Xdg Environment Variables Data Home

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The application requires a standardized mechanism for discovering configuration files across different deployment environments and user contexts
- Configuration file locations must be user-controllable without requiring code changes or recompilation
- The XDG Base Directory specification provides a widely-adopted standard for Unix-like systems to organize user-specific configuration, data, and cache files
- Evidence shows process.env.XDG_CONFIG_HOME is accessed in core configuration infrastructure modules (UnifiedConfigLoader, FileSystemUtils) with 0.89 significance across 2 files
- The pattern coordinates with Node.js filesystem operations and multiple configuration parsers (TOML, JSON) to resolve configuration paths at application startup

## Problem Statement

Applications need a consistent, portable mechanism to locate configuration files that respects user preferences and system conventions. Hardcoded configuration paths reduce portability and prevent users from organizing configuration files according to their environment's standards. Without a standardized approach, configuration discovery becomes fragmented across modules, leading to inconsistent behavior and deployment complexity.

## Decision

1. MAY: Additional XDG environment variables (XDG_DATA_HOME, XDG_CACHE_HOME) MAY be adopted for other resource types following the same pattern

## Policy Block

- MAY Additional XDG environment variables (XDG_DATA_HOME, XDG_CACHE_HOME) MAY be adopted for other resource types following the same pattern

In scope:
- All configuration file discovery and loading operations
- Core infrastructure modules responsible for application initialization
- Deployment scripts and documentation that reference configuration file locations
- User-facing configuration path specifications

Out of scope:
- Runtime configuration changes that do not involve file system access
- In-memory configuration overrides provided via command-line arguments or programmatic APIs
- Platform-specific configuration mechanisms on non-Unix systems where XDG is not applicable
- Embedded or containerized deployments where configuration is injected through alternative mechanisms

## Rationale

- The XDG Base Directory specification is a widely-adopted standard on Unix-like systems, providing predictable configuration file locations that users and system administrators expect
- Environment-variable-driven configuration path resolution enables deployment flexibility without code changes, supporting containerized, multi-user, and development environments
- Evidence shows this pattern is implemented in core configuration infrastructure (UnifiedConfigLoader, FileSystemUtils) with high significance (0.89), indicating it is a foundational architectural choice
- The pattern coordinates with multiple configuration formats (TOML via @iarna/toml, JSON) and filesystem operations, establishing a consistent abstraction layer for configuration discovery

## Consequences

Positive:
- Configuration files follow user and system conventions, improving user experience and reducing deployment friction
- Configuration paths are externally controllable without code modification, enabling environment-specific overrides
- Consistent configuration discovery pattern across all core modules reduces implementation fragmentation
- Alignment with XDG specification improves interoperability with other Unix tools and system management practices

Negative:
- Platform-specific behavior introduces complexity on Windows systems where XDG is not native
- Dependency on environment variables creates an external configuration surface that must be documented and validated
- Debugging configuration issues requires understanding both the XDG specification and the application's fallback logic
- Testing configuration discovery requires environment variable manipulation, complicating test setup

## Alternatives

- Use hardcoded configuration paths relative to application installation directory (rejected)
  Rejected because: Hardcoded paths prevent users from organizing configuration according to system conventions and complicate multi-user deployments
  When valid: Single-user, single-deployment scenarios where configuration portability is not required
- Search for configuration files in current working directory only (rejected)
  Rejected because: Current working directory is execution-context dependent and does not provide stable configuration locations across different invocation methods
  When valid: Development or testing scenarios where configuration is co-located with execution context
- Require explicit configuration file path via command-line argument (deferred)
  When valid: Can be used as an override mechanism in addition to XDG-based discovery for explicit user control

## Risks

- XDG_CONFIG_HOME may point to inaccessible or non-existent directories, causing configuration loading failures
  Mitigation: Implement robust error handling with clear error messages and fallback to default paths when XDG paths are invalid
  Owner: engineering team
- Windows and non-Unix platforms do not natively support XDG specification, requiring platform-specific adaptations
  Mitigation: Document platform-specific behavior and provide equivalent fallback mechanisms for non-Unix systems
  Owner: engineering team
- Users unfamiliar with XDG specification may not understand where configuration files should be placed
  Mitigation: Provide clear documentation with examples and diagnostic commands to show resolved configuration paths
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
- When XDG_CONFIG_HOME is undefined, implement fallback logic according to XDG specification defaults to ensure consistent behavior across environments
- Configuration path resolution should occur early in application initialization before any configuration-dependent modules are loaded
- Consider providing a diagnostic mode or logging option that displays resolved configuration paths to aid troubleshooting

## Continuation Context


Verify commands:
- Discover and execute the project's configuration validation test suite to verify XDG environment variable handling
- Inspect the core configuration modules to confirm process.env.XDG_CONFIG_HOME is accessed before fallback paths
- Run integration tests with XDG_CONFIG_HOME set to various values (valid path, invalid path, undefined) to verify error handling

Accept when:
- Configuration loading successfully resolves paths from XDG_CONFIG_HOME when the environment variable is set
- Configuration loading provides appropriate fallback behavior when XDG_CONFIG_HOME is undefined or inaccessible
- Error messages include resolved configuration paths to aid debugging

## Enforcement

- Verified by: Code review of configuration-related modules to ensure XDG environment variable checks are present
- Verified by: Integration tests that verify configuration discovery behavior under different XDG_CONFIG_HOME settings
- Verified by: Static analysis to detect direct filesystem path construction that bypasses XDG resolution
- Violation handling: Configuration modules that bypass XDG resolution must be refactored to use the established configuration infrastructure
- Violation handling: Pull requests introducing hardcoded configuration paths are rejected during code review
- Violation handling: Existing violations are tracked as technical debt and prioritized for remediation
- Exception process: Platform-specific code that requires alternative configuration mechanisms must document the rationale and provide equivalent functionality
- Exception process: Exceptions must be approved by the core infrastructure team and documented in the module
- Exception process: Temporary exceptions during migration are permitted with a defined remediation timeline