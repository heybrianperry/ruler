# Adopt XDG Base Directory Specification for Configuration File Location: Error Handling Around Configuration Directory Access

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application requires a predictable, cross-platform mechanism for locating user and system configuration files without hardcoding paths
- Configuration loading spans multiple subsystems including CLI handlers, file system utilities, and configuration parsers, requiring a consistent path resolution strategy
- The XDG Base Directory specification provides a standardized approach for Unix-like systems to separate user configuration from application code and system defaults
- Evidence shows process.env.XDG_CONFIG_HOME accessed across CLI handlers, file system utilities, and configuration loading modules with significance 0.89

## Problem Statement

The application needs a standardized, user-controllable mechanism for locating configuration files across different execution environments and platforms. Hardcoded paths reduce portability and prevent users from organizing configuration according to their system conventions. Without a consistent specification, different modules may resolve configuration paths inconsistently, leading to fragmented configuration loading logic.

## Decision

1. SHOULD: Error handling around configuration directory access SHOULD log the resolved path to aid debugging when XDG environment variables are misconfigured

## Policy Block

- SHOULD Error handling around configuration directory access SHOULD log the resolved path to aid debugging when XDG environment variables are misconfigured

In scope:
- All configuration file loading operations at application startup
- CLI handlers that initialize configuration subsystems
- File system utilities that resolve user or system configuration paths
- Configuration parsers that need to locate configuration files before reading them

Out of scope:
- Runtime-generated temporary files or cache files (unless explicitly using XDG_CACHE_HOME)
- Application-internal resource files bundled with the distribution
- Configuration passed via command-line arguments or standard input
- Platform-specific system configuration paths that do not follow XDG conventions (e.g., Windows registry)

## Rationale

- The XDG Base Directory specification is a widely adopted standard on Unix-like systems, providing predictable configuration file locations that users can control via environment variables
- Evidence shows consistent adoption across CLI handlers, file system utilities, and configuration loading with process.env.XDG_CONFIG_HOME access in 3 files at significance 0.89
- Separating configuration location from application logic improves portability and allows users to organize configuration according to their system conventions
- Integration with Node.js path and os modules enables cross-platform fallback behavior when XDG variables are not set

## Consequences

Positive:
- Configuration files follow a standardized, predictable location scheme that users can control via environment variables
- Improved portability across Unix-like systems by adhering to a widely recognized specification
- Separation of user configuration from application code and system defaults, supporting multi-user environments
- Consistent path resolution logic across CLI, file system utilities, and configuration loading modules

Negative:
- XDG Base Directory specification is primarily Unix-centric; Windows and macOS have different native conventions requiring additional fallback logic
- Developers must understand XDG environment variables and their precedence to debug configuration loading issues
- Testing configuration loading requires setting environment variables or mocking process.env, adding complexity to test setup
- Users unfamiliar with XDG conventions may not know where configuration files are located without consulting documentation

## Alternatives

- Hardcode configuration paths relative to application installation directory or user home directory (rejected)
  Rejected because: Reduces portability, prevents users from organizing configuration according to system conventions, and creates inconsistency with other XDG-compliant applications on Unix-like systems
  When valid: In single-user, single-platform applications where configuration location is not a user concern
- Use platform-specific native configuration APIs (Windows registry, macOS preferences, Linux XDG) (rejected)
  Rejected because: Increases implementation complexity with platform-specific code paths; evidence shows Node.js process.env approach is sufficient for current requirements
  When valid: When deep integration with platform-native configuration systems is required, such as system-wide settings or OS-managed preferences
- Require configuration file path to be explicitly provided via command-line argument or environment variable (rejected)
  Rejected because: Places burden on users to specify paths for every invocation; XDG provides sensible defaults while still allowing override via XDG_CONFIG_HOME
  When valid: In containerized or CI/CD environments where explicit configuration paths are preferred over convention-based discovery

## Risks

- XDG_CONFIG_HOME may be unset or point to a non-existent directory, causing configuration loading to fail without clear error messages
  Mitigation: Implement robust fallback logic following XDG specification defaults; log resolved paths during configuration loading; validate directory existence and permissions with informative error messages
  Owner: engineering team
- Windows and macOS do not natively support XDG Base Directory specification, potentially causing confusion or requiring platform-specific fallback paths
  Mitigation: Document platform-specific behavior; implement fallback to platform-appropriate defaults when XDG variables are unset; test configuration loading on all target platforms
  Owner: engineering team
- Multiple modules independently accessing process.env.XDG_CONFIG_HOME may resolve paths inconsistently if fallback logic differs
  Mitigation: Centralize XDG path resolution in a shared utility module; ensure all configuration loading paths use the same resolution function; add integration tests verifying consistent path resolution across modules
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
- Centralize XDG path resolution logic in a shared utility module to ensure consistent behavior across CLI handlers, file system utilities, and configuration loaders
- Implement fallback logic that follows XDG Base Directory specification defaults when XDG_CONFIG_HOME is unset, typically defaulting to a subdirectory within the user home directory
- Log the resolved configuration directory path during initialization to aid debugging when users have custom XDG environment variable configurations

## Continuation Context


Verify commands:
- Discover the project test suite and execute tests covering configuration loading with XDG_CONFIG_HOME set to a custom path
- Discover the project test suite and execute tests covering configuration loading with XDG_CONFIG_HOME unset to verify fallback behavior
- Inspect the codebase for all modules accessing environment variables for configuration paths and verify they use the centralized XDG-aware resolution mechanism

Accept when:
- All configuration loading tests pass with both XDG_CONFIG_HOME set and unset
- Configuration path resolution is consistent across CLI handlers, file system utilities, and configuration loading modules
- Error messages clearly indicate the resolved configuration directory path when configuration loading fails

## Enforcement

- Verified by: Code review verifying all configuration path resolution uses the centralized XDG-aware utility
- Verified by: Integration tests covering configuration loading with various XDG environment variable configurations
- Verified by: Static analysis detecting direct hardcoded configuration paths that bypass XDG resolution
- Violation handling: Code review feedback requiring refactoring to use centralized XDG path resolution
- Violation handling: CI pipeline failure if integration tests detect inconsistent configuration path resolution
- Violation handling: Documentation updates required when new configuration loading paths are added
- Exception process: Platform-specific configuration paths that cannot follow XDG conventions require explicit documentation and architectural review
- Exception process: Temporary files or cache files may use alternative location strategies if documented and scoped appropriately
- Exception process: Exceptions must be documented in code comments explaining why XDG path resolution is not applicable