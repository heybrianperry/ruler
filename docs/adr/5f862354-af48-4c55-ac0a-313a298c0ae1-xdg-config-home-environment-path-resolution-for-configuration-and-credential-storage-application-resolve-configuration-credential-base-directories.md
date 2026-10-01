# XDG_CONFIG_HOME Environment Path Resolution for Configuration and Credential Storage: Application Resolve Configuration Credential Base Directories

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Application components must locate global configuration files, user preferences, and credential storage directories across diverse operating environments.
- Hardcoded filesystem paths create brittle execution behavior in containerized environments and multi-user systems.
- Static analysis identifies consistent access to process.env.XDG_CONFIG_HOME across CLI handlers, filesystem utilities, and configuration loading modules.

## Problem Statement

CLI tools and runtime configuration loaders require a consistent, platform-compliant strategy to discover the base filesystem location for user configuration and credential storage without hardcoding user home paths or breaking isolation in multi-tenant and containerized environments.

## Decision

1. MUST: The application MUST resolve configuration and credential base directories using process.env.XDG_CONFIG_HOME before falling back to default user directory locations.

## Policy Block

- MUST The application MUST resolve configuration and credential base directories using process.env.XDG_CONFIG_HOME before falling back to default user directory locations.

In scope:
- Resolution of global configuration directories and credential storage locations across CLI command handlers and runtime services.
- Storage, retrieval, and validation of user settings and authentication tokens.

Out of scope:
- Local repository-scoped configuration files that do not involve user-level or system-wide credential persistence.
- Ephemeral in-memory configuration passed directly via invocation parameters.

## Rationale

- Conforming to process.env.XDG_CONFIG_HOME provides standard base directory resolution across operating environments, allowing users and automated agents to relocate configuration and secrets.
- Centralizing directory derivation prevents divergent path logic and inconsistent credential access across CLI handlers and configuration loaders.
- Dynamic environment inspection ensures compatibility with containerized environments and multi-user configurations without hardcoding filesystem paths.

## Consequences

Positive:
- Standardizes configuration and credential storage locations across operating systems according to established desktop and server conventions.
- Enables testing and container isolation by redirecting configuration directories through environment configuration.
- Prevents accidental hardcoding of user paths in application handlers and loaders.

Negative:
- Introduces reliance on runtime environment variables that may differ between interactive and non-interactive execution contexts.
- Requires defensive error handling and permission verification when resolved environment paths are invalid or inaccessible.

## Alternatives

- Hardcoded User Home Directory Paths (rejected)
  Rejected because: Inflexible across differing operating system conventions and prevents custom credential path redirection in containerized or test environments.
  When valid: Minimal single-platform internal prototypes with no requirement for environment-based path configuration.
- System-Wide Keyring or Dedicated Secret Daemon Integration (deferred)
  Rejected because: Adds heavy native dependency requirements and operating-system-specific compilation constraints across supported platforms.
  When valid: Environments where operating-system-level hardware-backed secret storage is strictly mandated by compliance policies.

## Risks

- Distributed reads of process.env.XDG_CONFIG_HOME across multiple modules can cause path divergence if fallback handling differs.
  Mitigation: Encapsulate path resolution within a shared core filesystem utility that all modules must consume.
  Owner: engineering team
- An invalid or inaccessible directory configured in the environment variable may trigger unhandled filesystem exceptions.
  Mitigation: Validate directory existence and read/write permissions at resolution time with structured fallback behavior.
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
- Centralize base directory resolution within dedicated core path utilities to avoid duplicating environment inspection logic.
- Sanitize error logging output to ensure directory resolution failures do not expose sensitive environment tokens or credential values.

## Continuation Context


Verify commands:
- Discover and execute the project unit test suite for configuration and path resolution utilities.
- Discover and execute the repository static analysis and linting scripts to verify adherence to centralized path utilities.

Accept when:
- Configuration path resolution tests pass, demonstrating correct precedence of process.env.XDG_CONFIG_HOME over default user directory fallbacks.
- Static analysis confirms no unencapsulated reads of the environment variable exist outside core filesystem utility modules.

## Enforcement

- Verified by: Automated continuous integration pipeline testing configuration resolution suites.
- Verified by: Peer code review verifying path resolution and secret handling conventions.
- Violation handling: Continuous integration failure for unencapsulated environment variable access or broken resolution tests.
- Violation handling: Code review rejection for hardcoded user paths or unmasked credential logging.
- Exception process: Submit an architectural review request documented in repository tracking with lead engineer approval.