# Adopt @iarna/toml for TOML Configuration Parsing: Before Implementing Any Toml Parsing Logic

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The application uses TOML as a configuration file format across agent definitions, MCP server configurations, and application-level settings
- Configuration parsing occurs in multiple modules including agent initialization (CodexCliAgent, MistralVibeAgent), core configuration loading (ConfigLoader), subagent processing (SubagentsProcessor), and MCP integration (propagateOpenHandsMcp)
- TOML parsing requires a third-party library in JavaScript/TypeScript ecosystems as no native parser exists in the runtime
- The codebase has standardized on a single TOML parsing library to ensure consistent parsing behavior and reduce dependency surface area
- Parsed TOML content feeds into validation schemas and runtime configuration objects that drive application behavior

## Problem Statement

The application requires a reliable, consistent mechanism to parse TOML configuration files across multiple modules. Without a standardized TOML parsing library, different modules might adopt incompatible parsers leading to inconsistent parsing behavior, increased dependency complexity, and maintenance burden. A single library choice ensures uniform TOML interpretation across the configuration layer.

## Decision

1. MUST: Before implementing any TOML parsing logic, developers MUST discover the project's dependency lock artifact and resolve the exact installed version of the TOML parsing library

## Policy Block

- MUST Before implementing any TOML parsing logic, developers MUST discover the project's dependency lock artifact and resolve the exact installed version of the TOML parsing library

In scope:
- All modules that load TOML configuration files from the filesystem
- Agent initialization and configuration loading logic
- MCP server configuration processing
- Application-level configuration management
- Subagent processor configuration parsing

Out of scope:
- Runtime configuration that does not originate from TOML files
- JSON, YAML, or other configuration format parsing
- Environment variable processing
- Command-line argument parsing
- Configuration generation or serialization to TOML format (if different library is more appropriate for writing)

## Rationale

- Evidence shows the TOML parsing library is consistently imported and used across 5 files spanning agents, core configuration, and MCP integration with pattern significance of 0.92
- Standardizing on a single TOML parsing library eliminates inconsistencies in TOML interpretation, reduces the dependency footprint, and simplifies maintenance
- The library integrates cleanly with the existing file system operations pattern (fs/fs.promises + path) and validation layer (zod schemas observed in ConfigLoader)
- Centralizing TOML parsing on one library enables consistent error handling and debugging across the configuration layer

## Consequences

Positive:
- Consistent TOML parsing behavior across all configuration files ensures predictable application behavior
- Single dependency for TOML parsing reduces bundle size and simplifies dependency management
- Developers have a clear, documented standard for implementing new configuration file parsing
- Centralized parsing library choice enables easier migration or upgrade if parsing issues arise

Negative:
- The project is coupled to the chosen library's parsing behavior, quirks, and maintenance status
- If the library has parsing bugs or incompatibilities with TOML specification updates, all configuration parsing is affected
- Switching to an alternative TOML library in the future requires changes across 5+ files
- Developers cannot choose alternative TOML parsers even if they have experience with different libraries

## Alternatives

- Use multiple TOML parsing libraries based on module-specific needs (rejected)
  Rejected because: Multiple libraries increase dependency surface area, create inconsistent parsing behavior across modules, and complicate maintenance
  When valid: Never valid for this codebase given the need for consistent configuration interpretation
- Implement a custom TOML parser (rejected)
  Rejected because: Custom parser implementation requires significant development effort, ongoing maintenance, and TOML specification compliance testing that a mature library already provides
  When valid: Only if no existing library meets security, performance, or compatibility requirements
- Migrate configuration format from TOML to JSON or YAML with native/standard parsing (rejected)
  Rejected because: Evidence shows TOML is already established across the configuration layer; migration would require rewriting all configuration files and parsing logic
  When valid: If TOML parsing becomes a significant maintenance burden or if the format proves inadequate for configuration needs

## Risks

- The chosen TOML library may become unmaintained or have security vulnerabilities
  Mitigation: Monitor library maintenance status and security advisories; maintain test coverage for TOML parsing to enable library migration if needed
  Owner: engineering team
- Library may have parsing incompatibilities with TOML specification updates or edge cases
  Mitigation: Validate parsed configuration against schemas; include TOML parsing tests in CI; document any known parsing limitations
  Owner: engineering team
- Developers unfamiliar with the library's API may implement incorrect error handling or parsing patterns
  Mitigation: Document standard parsing patterns; provide code examples in implementation notes; enforce code review for configuration parsing changes
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
- Standard parsing pattern: read file content as string using filesystem APIs, then pass the string to the TOML library's parse function, then validate the resulting object against a schema before consuming configuration values
- Error handling should distinguish between file system errors (file not found, permission denied) and TOML parsing errors (invalid syntax, type mismatches) to provide actionable error messages
- When adding new configuration files, ensure TOML syntax is validated during development and that parsing errors are caught in CI before deployment

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and identify the TOML parsing library entry
- Locate and inspect the dependency lock artifact to confirm the TOML library is installed at a specific resolved version
- Search the codebase for imports of the TOML parsing library and verify all usage follows the standard parsing pattern
- Execute the project's test suite and verify TOML parsing tests pass with the installed library version

Accept when:
- All TOML configuration parsing uses the adopted library's API functions
- No alternative TOML parsing libraries are present in the dependency manifest
- TOML parsing operations include error handling for invalid syntax
- Test coverage exists for TOML configuration parsing across affected modules

## Enforcement

- Verified by: Code review verification that new TOML parsing code uses the adopted library
- Verified by: Dependency audit in CI to detect introduction of alternative TOML parsing libraries
- Verified by: Static analysis to verify imports match the adopted library
- Violation handling: Pull requests introducing alternative TOML libraries are rejected with reference to this ADR
- Violation handling: Code using non-standard TOML parsing patterns is flagged in review and must be refactored
- Violation handling: CI fails if dependency manifest contains multiple TOML parsing libraries
- Exception process: Exceptions require architectural review and must document specific technical justification
- Exception process: Exception requests must demonstrate why the adopted library cannot meet the requirement
- Exception process: Approved exceptions are documented as ADR amendments with scope limitations