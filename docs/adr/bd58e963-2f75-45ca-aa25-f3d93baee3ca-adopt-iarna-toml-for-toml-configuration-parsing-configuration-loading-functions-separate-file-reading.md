# Adopt @iarna/toml for TOML Configuration Parsing: Configuration Loading Functions Separate File Reading

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The project uses TOML as a configuration file format for agent definitions, MCP server configurations, and core application settings
- Configuration loading occurs in multiple infrastructure components including core config loaders, agent initialization systems, and MCP propagation logic
- TOML parsing is integrated with validation pipelines using Zod schemas to ensure type safety and structural correctness of loaded configurations
- The configuration system reads TOML files from the filesystem and requires a reliable parser to convert TOML syntax into JavaScript objects before validation

## Problem Statement

The project requires a TOML parsing library to load configuration files across multiple infrastructure components. The parser must reliably convert TOML syntax into JavaScript objects that can be validated and consumed by the configuration system, agent initialization logic, and MCP server management.

## Decision

1. SHOULD: Configuration loading functions SHOULD separate file reading, TOML parsing, and validation into distinct steps for clarity and error handling

## Policy Block

- SHOULD Configuration loading functions SHOULD separate file reading, TOML parsing, and validation into distinct steps for clarity and error handling

In scope:
- Configuration file loading in core infrastructure components
- Agent definition file parsing
- MCP server configuration parsing
- Any component that reads TOML files from the filesystem for application configuration

Out of scope:
- Runtime configuration that does not originate from TOML files
- Configuration formats other than TOML such as JSON or YAML
- TOML generation or serialization (this ADR covers parsing only)
- User-provided TOML content that is not part of the application's configuration system

## Rationale

- The @iarna/toml library is consistently used across three core infrastructure files (ConfigLoader, CodexCliAgent, propagateOpenHandsMcp) with high significance scores (0.89-0.93), indicating an established architectural choice
- TOML provides a human-readable configuration format with strong typing semantics that aligns with the project's use of Zod validation schemas for type safety
- Standardizing on a single TOML parsing library ensures consistent parsing behavior across all configuration loading paths and reduces dependency complexity
- The integration pattern of parse-then-validate separates syntax processing from semantic validation, enabling clear error messages and modular configuration handling

## Consequences

Positive:
- Consistent TOML parsing behavior across all configuration loading components
- Single dependency for TOML parsing reduces bundle size and maintenance surface area
- Clear separation between parsing and validation enables better error handling and debugging
- Established usage pattern across multiple files provides implementation examples for new configuration loading code

Negative:
- Dependency on a third-party library introduces maintenance risk if the library becomes unmaintained or has security vulnerabilities
- Switching to a different TOML parser would require changes across multiple infrastructure components
- Parse errors from @iarna/toml may not provide optimal error messages for end users without additional wrapping
- The library's API and performance characteristics constrain how TOML parsing can be implemented throughout the codebase

## Alternatives

- Use the built-in JSON.parse with JSON configuration files instead of TOML (rejected)
  Rejected because: TOML provides better human readability for configuration files and supports comments, which are valuable for documenting configuration options. The project has already adopted TOML as evidenced by existing configuration files.
  When valid: For new projects that have not yet committed to a configuration format and prefer JSON's ubiquity
- Use an alternative TOML parsing library from the ecosystem (rejected)
  Rejected because: The evidence shows @iarna/toml is already integrated across three core components with high significance. Switching would require refactoring established code without clear benefit.
  When valid: If @iarna/toml becomes unmaintained, has critical security issues, or demonstrates significant performance problems
- Implement a custom TOML parser (rejected)
  Rejected because: TOML parsing is complex and error-prone. Using a well-tested library reduces implementation risk and maintenance burden compared to maintaining a custom parser.
  When valid: Never recommended unless the project has extremely specific parsing requirements that no existing library can satisfy

## Risks

- The @iarna/toml library may become unmaintained or have security vulnerabilities discovered
  Mitigation: Monitor dependency security advisories and maintain awareness of alternative TOML parsing libraries. Document the migration path to an alternative parser if needed.
  Owner: Engineering team
- Parse errors may not provide sufficient context for users to debug malformed TOML configuration files
  Mitigation: Wrap parse errors with additional context about which file failed and provide guidance on TOML syntax. Consider implementing configuration validation that provides user-friendly error messages.
  Owner: Engineering team
- Performance characteristics of @iarna/toml may not scale well for very large configuration files
  Mitigation: Monitor configuration file sizes and parsing performance. If performance issues arise, evaluate alternative parsers or consider splitting large configuration files into smaller modules.
  Owner: Engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Examine existing configuration loading implementations in the core config loader, agent initialization, and MCP propagation modules to understand the established parse-then-validate pattern
- When adding new configuration file loading, follow the pattern of reading file content, parsing with @iarna/toml, then validating the resulting object with appropriate schemas
- Wrap parse operations in try-catch blocks and enrich error messages with the configuration file path and guidance for users to debug TOML syntax errors

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and verify that @iarna/toml is declared as a dependency
- Discover and execute the project's test suite to verify that configuration loading tests pass with the current TOML parser implementation
- Search the codebase for all imports of @iarna/toml and verify they follow the established parse-then-validate pattern

Accept when:
- All TOML configuration parsing uses @iarna/toml library
- Configuration loading tests pass and demonstrate proper error handling for malformed TOML
- No alternative TOML parsing libraries are imported or used in configuration loading code paths

## Enforcement

- Verified by: Code review verification that new configuration loading code uses @iarna/toml
- Verified by: Dependency analysis to detect introduction of alternative TOML parsing libraries
- Verified by: Integration tests that verify configuration loading behavior across all components
- Violation handling: Code review feedback requesting alignment with the established TOML parsing library
- Violation handling: Pull request comments explaining the rationale for standardizing on @iarna/toml
- Violation handling: Refactoring guidance to migrate any non-compliant code to use the standard library
- Exception process: Document the specific technical requirement that cannot be satisfied by @iarna/toml
- Exception process: Propose an alternative approach and evaluate its impact on the existing configuration loading architecture
- Exception process: Obtain approval from the engineering team lead before introducing an alternative TOML parser