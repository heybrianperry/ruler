# Standardize XDG_CONFIG_HOME for User-Level Configuration Discovery: Implementations Extend Xdg

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires a mechanism to locate user-level configuration directories across different operating systems and user environments
- Two modules (src/core/FileSystemUtils.ts and src/cli/handlers.ts) independently access process.env.XDG_CONFIG_HOME to resolve global configuration paths
- The XDG Base Directory specification provides a cross-platform standard for user configuration file locations, avoiding hardcoded paths
- Configuration discovery must handle cases where XDG_CONFIG_HOME is undefined and fall back to appropriate defaults
- The pattern appears in both core filesystem utilities and CLI handler logic, indicating a cross-cutting concern for configuration management

## Problem Statement

The system needs a consistent, cross-platform approach to discover user-level configuration directories without hardcoding paths or creating platform-specific branches throughout the codebase. Without a standardized environment variable convention, configuration discovery logic becomes fragmented, error-prone, and difficult to test across different deployment environments.

## Decision

1. MAY: Implementations MAY extend XDG_CONFIG_HOME-based discovery with additional platform-specific fallbacks for compatibility

## Policy Block

- MAY Implementations MAY extend XDG_CONFIG_HOME-based discovery with additional platform-specific fallbacks for compatibility

In scope:
- User-level configuration file discovery in src/core/FileSystemUtils.ts
- CLI handler configuration resolution in src/cli/handlers.ts
- Global configuration directory lookups that span user environments
- Runtime environment variable access for configuration paths

Out of scope:
- Project-local configuration files within the working directory
- System-wide configuration in /etc or equivalent system directories
- Temporary or cache directories (covered by XDG_CACHE_HOME or XDG_RUNTIME_DIR)
- Application data directories (covered by XDG_DATA_HOME)

Exceptions:
- EXC-001: Platform-specific requirements mandate alternative configuration paths (e.g., Windows registry, macOS Application Support)

## Rationale

- The XDG Base Directory specification is widely adopted in Unix-like systems and provides predictable, user-controllable configuration paths
- Evidence shows process.env.XDG_CONFIG_HOME is accessed in both core utilities and CLI handlers, indicating an established pattern with 89.25% confidence across 2 files
- Using environment variables for configuration discovery enables testing, containerization, and multi-user deployments without code changes
- Centralizing on XDG_CONFIG_HOME reduces platform-specific branching and aligns with POSIX conventions while remaining extensible for other platforms

## Consequences

Positive:
- Configuration paths become predictable and user-controllable through standard environment variables
- Testing and CI environments can override configuration locations without modifying code
- Alignment with XDG specification improves interoperability with other Unix-like tools and package managers
- Reduced hardcoded paths improve portability across different deployment environments

Negative:
- Dependency on environment variables requires documentation and may confuse users unfamiliar with XDG conventions
- Windows environments require additional fallback logic since XDG_CONFIG_HOME is not native to that platform
- Runtime behavior becomes dependent on external environment state, complicating debugging when misconfigured
- Multiple modules accessing process.env directly creates coupling to Node.js runtime environment

## Alternatives

- Hardcode configuration paths to ~/.config or platform-specific defaults (rejected)
  Rejected because: Hardcoded paths prevent user customization, complicate testing, and violate the principle of environment-driven configuration
  When valid: Only in fallback scenarios when XDG_CONFIG_HOME is undefined
- Use a configuration management library (e.g., cosmiconfig) to abstract path resolution (deferred)
  Rejected because: Adds external dependency; current evidence shows direct environment variable access is sufficient for the pattern's scope
  When valid: If configuration discovery requirements expand to include multiple formats, cascading configs, or complex search paths
- Implement platform-specific path resolution for each OS (Windows, macOS, Linux) (rejected)
  Rejected because: Increases code complexity and maintenance burden; XDG_CONFIG_HOME with fallbacks provides sufficient cross-platform coverage
  When valid: If native platform integration becomes a hard requirement for specific deployment targets

## Risks

- Users may not set XDG_CONFIG_HOME, causing configuration discovery to fail or fall back to unexpected defaults
  Mitigation: Implement robust fallback logic (e.g., ~/.config) and document the environment variable requirement in user-facing documentation
  Owner: Engineering team
- Direct access to process.env in multiple modules creates scattered configuration logic that is difficult to test and maintain
  Mitigation: Refactor to centralize environment variable access in a dedicated configuration module with clear public contracts
  Owner: Engineering team
- Security vulnerabilities if XDG_CONFIG_HOME points to untrusted or world-writable directories
  Mitigation: Validate resolved paths for symbolic links and permissions before reading configuration files, as evidenced by assertNotSymbolicLink usage
  Owner: Security team

## Implementation Notes

- Centralize XDG_CONFIG_HOME access in a dedicated configuration utility module to reduce duplication between FileSystemUtils.ts and handlers.ts
- Implement fallback logic: if XDG_CONFIG_HOME is undefined, default to path.join(os.homedir(), '.config') on POSIX systems
- Add validation for resolved configuration directories: check for symbolic links, verify containing directory is within expected root, and log errors with full context
- Document the XDG_CONFIG_HOME requirement in README and CLI help text, including examples for setting the variable in different shells

## Continuation Context


Verify commands:
- grep -r 'process\.env\.XDG_CONFIG_HOME' src/ | wc -l
- grep -r 'XDG_CONFIG_HOME' src/ --include='*.ts' -A 3 | grep -E '(fallback|default|undefined)'
- npm test -- --grep 'XDG_CONFIG_HOME'

Accept when:
- All modules that require user-level configuration paths access process.env.XDG_CONFIG_HOME with documented fallback behavior
- Error logging for configuration directory access includes the resolved path and error details
- Tests verify configuration discovery behavior with XDG_CONFIG_HOME set, unset, and pointing to invalid paths

## Enforcement

- Verified by: Static analysis to detect direct process.env access outside approved configuration modules
- Verified by: Code review checklist requiring XDG_CONFIG_HOME usage for user-level configuration paths
- Verified by: Integration tests validating configuration discovery across different environment variable states
- Violation handling: CI pipeline fails if new code accesses hardcoded configuration paths instead of XDG_CONFIG_HOME
- Violation handling: Pull requests introducing configuration path logic require architecture review
- Violation handling: Existing violations are tracked as technical debt items with prioritized remediation
- Exception process: Document platform-specific constraints requiring alternative configuration mechanisms
- Exception process: Obtain approval from architecture review board with justification
- Exception process: Maintain XDG_CONFIG_HOME as the primary mechanism and isolate exceptions to platform-specific modules