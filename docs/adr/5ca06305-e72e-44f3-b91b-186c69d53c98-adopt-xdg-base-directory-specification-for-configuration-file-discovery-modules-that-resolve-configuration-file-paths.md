# Adopt XDG Base Directory Specification for Configuration File Discovery: Modules That Resolve Configuration File Paths

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application requires a standardized, cross-platform mechanism to locate user and system configuration files without hardcoding paths
- Configuration file discovery must respect user preferences and system conventions for config directory placement
- The XDG Base Directory specification provides a widely-adopted standard for Unix-like systems to organize user-specific configuration, data, and cache files
- Evidence shows consistent usage of process.env.XDG_CONFIG_HOME across CLI handlers, configuration loading, and filesystem utilities, indicating a deliberate architectural choice

## Problem Statement

The application needs to discover configuration files in a predictable, user-controllable location that follows platform conventions. Hardcoding configuration paths reduces portability and prevents users from organizing their configuration directories according to their preferences or system policies. Without a standardized approach, different modules might resolve configuration locations inconsistently, leading to fragmented configuration management.

## Decision

1. MUST: All modules that resolve configuration file paths MUST use a consistent mechanism to read XDG_CONFIG_HOME from the process environment

## Policy Block

- MUST All modules that resolve configuration file paths MUST use a consistent mechanism to read XDG_CONFIG_HOME from the process environment

In scope:
- All configuration file discovery and loading operations
- CLI initialization that requires access to user or system configuration
- Filesystem utilities that resolve global or user-specific configuration directories
- Any module that needs to determine where configuration files are stored

Out of scope:
- Project-local configuration files that are stored within the working directory
- Embedded or compiled-in default configurations
- Configuration passed via command-line arguments or flags
- Runtime configuration that does not persist to the filesystem

## Rationale

- The XDG Base Directory specification is a widely-adopted standard on Unix-like systems, providing predictable configuration file locations that users and system administrators expect
- Using environment variables for configuration discovery enables users to override default locations without modifying application code, supporting diverse deployment environments
- Consistent usage across CLI handlers, configuration loaders, and filesystem utilities (3 files) demonstrates this is an intentional architectural pattern rather than isolated implementation detail
- The pattern separates configuration location concerns from configuration content validation, as evidenced by Zod schemas handling structure while XDG_CONFIG_HOME handles discovery

## Consequences

Positive:
- Configuration files follow platform conventions, improving user experience and system integration
- Users can relocate configuration directories by setting a single environment variable without application changes
- The application integrates cleanly with system management tools and dotfile managers that respect XDG conventions
- Configuration discovery logic is centralized and consistent across all modules that need it

Negative:
- The application depends on runtime environment variables, which may not be set in all execution contexts
- XDG Base Directory specification is primarily Unix-focused, requiring additional platform-specific handling for Windows
- Configuration discovery behavior becomes less transparent to developers unfamiliar with XDG conventions
- Testing requires environment variable manipulation to verify configuration discovery paths

## Alternatives

- Hardcode configuration directory paths relative to the application installation directory (rejected)
  Rejected because: Hardcoded paths prevent users from organizing configuration according to their preferences and violate platform conventions for user-specific configuration storage
  When valid: Only appropriate for embedded systems or single-user appliances where configuration location is fixed by design
- Use a custom environment variable specific to this application for configuration directory location (rejected)
  Rejected because: Custom environment variables fragment the configuration ecosystem and prevent integration with system tools that manage XDG-compliant applications
  When valid: May be used as a secondary override mechanism in addition to XDG_CONFIG_HOME for application-specific customization
- Search multiple predefined locations in priority order without environment variable consultation (rejected)
  Rejected because: Search-based discovery is less predictable and prevents explicit user control over configuration location, though it may be used as a fallback mechanism
  When valid: Appropriate as a fallback when XDG_CONFIG_HOME is not set, following XDG specification defaults

## Risks

- XDG_CONFIG_HOME may not be set in all execution environments, causing configuration discovery to fail
  Mitigation: Implement fallback logic following XDG specification defaults and document environment requirements
  Owner: engineering team
- Platform-specific differences in environment variable handling may cause inconsistent behavior across operating systems
  Mitigation: Test configuration discovery on all supported platforms and implement platform-specific adaptations where necessary
  Owner: engineering team
- Changes to XDG_CONFIG_HOME during application runtime may cause inconsistent configuration resolution
  Mitigation: Resolve configuration directory location during initialization and cache the result for the application lifecycle
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
- Implement fallback logic that uses the XDG Base Directory specification default when XDG_CONFIG_HOME is not set, typically resolving to a subdirectory within the user's home directory
- Centralize XDG_CONFIG_HOME resolution in a single utility module to ensure consistent behavior across all configuration loading code paths
- Consider caching the resolved configuration directory path during application initialization to avoid repeated environment variable lookups and ensure consistent behavior throughout the application lifecycle

## Continuation Context


Verify commands:
- Discover the project's test suite and execute tests covering configuration directory resolution with various XDG_CONFIG_HOME values
- Locate and run integration tests that verify configuration loading behavior when XDG_CONFIG_HOME is set, unset, or points to non-existent directories
- Identify static analysis or linting rules that enforce consistent environment variable access patterns across configuration-related modules

Accept when:
- All configuration loading tests pass with XDG_CONFIG_HOME set to various valid and invalid values
- Configuration directory resolution produces consistent results across all modules that implement it
- Error handling gracefully manages cases where XDG_CONFIG_HOME points to inaccessible or non-existent directories

## Enforcement

- Verified by: Code review verifying that new configuration loading code consults XDG_CONFIG_HOME
- Verified by: Automated tests covering configuration discovery with various environment variable states
- Verified by: Static analysis detecting direct filesystem path construction that bypasses XDG resolution
- Violation handling: Code review feedback requiring refactoring to use XDG-based configuration discovery
- Violation handling: Test failures when configuration loading does not respect XDG_CONFIG_HOME
- Violation handling: Documentation updates required for any approved exceptions to the XDG pattern
- Exception process: Document the specific use case requiring an exception to XDG-based configuration discovery
- Exception process: Obtain approval from the architecture review team for configuration patterns that bypass XDG conventions
- Exception process: Record the exception rationale and alternative approach in code comments and architecture documentation