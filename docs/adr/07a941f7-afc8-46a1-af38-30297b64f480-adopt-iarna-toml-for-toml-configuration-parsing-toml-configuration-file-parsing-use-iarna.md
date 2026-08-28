# Adopt @iarna/toml for TOML Configuration Parsing: Toml Configuration File Parsing Use Iarna

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The project uses TOML as the configuration file format for agent definitions, system configuration, and MCP server specifications
- Configuration loading occurs across multiple subsystems including agent initialization (CodexCliAgent, MistralVibeAgent), core configuration management (ConfigLoader, UnifiedConfigLoader), subsystem processors (SubagentsProcessor), and MCP server propagation
- TOML parsing is a prerequisite step before validation (Zod schemas) and configuration application, requiring a reliable and consistent parsing implementation
- The codebase demonstrates consistent adoption of @iarna/toml across 6 files spanning configuration loading, agent initialization, and MCP management contexts with significance scores 0.89-0.93

## Problem Statement

The project requires a standardized approach to parsing TOML configuration files across multiple subsystems. Without a consistent TOML parsing library, different modules might adopt incompatible parsers leading to inconsistent parsing behavior, maintenance burden from multiple dependencies, and potential configuration compatibility issues between subsystems.

## Decision

1. MUST: All TOML configuration file parsing MUST use the @iarna/toml library

## Policy Block

- MUST All TOML configuration file parsing MUST use the @iarna/toml library

In scope:
- All configuration file loading where the source format is TOML
- Agent configuration initialization and updates
- System configuration loading (ConfigLoader, UnifiedConfigLoader)
- MCP server definition parsing
- Any subsystem that reads TOML-formatted files from the filesystem

Out of scope:
- Runtime data serialization or deserialization (use appropriate runtime format)
- Non-TOML configuration formats such as JSON or YAML
- TOML generation or serialization (this ADR governs parsing only)
- Configuration validation logic (handled by separate validation libraries)

## Rationale

- Standardizing on a single TOML parsing library ensures consistent parsing behavior across all configuration subsystems and eliminates compatibility issues between modules
- The @iarna/toml library is already adopted across 6 files spanning agent initialization, configuration loading, and MCP management with high significance (0.89-0.93), indicating established usage and proven reliability in the codebase
- Consolidating TOML parsing to one library reduces dependency footprint, simplifies maintenance, and provides a single point of upgrade for TOML specification compliance
- The observed pattern of parsing before validation (Zod schemas in ConfigLoader) demonstrates architectural separation of concerns that this standardization reinforces

## Consequences

Positive:
- Consistent TOML parsing behavior across all configuration subsystems eliminates subtle compatibility issues
- Single dependency for TOML parsing reduces maintenance burden and simplifies security audits
- Clear standard simplifies onboarding for developers implementing new configuration loading features
- Centralized parsing library enables consistent error handling and debugging across configuration contexts

Negative:
- Dependency on a single library creates risk if the library is abandoned or has critical bugs
- Migration cost if @iarna/toml proves inadequate for future requirements
- Potential performance or feature limitations if alternative TOML parsers offer superior capabilities
- Lock-in to library-specific parsing behavior may complicate future TOML specification compliance

## Alternatives

- Use the 'toml' npm package instead of @iarna/toml (rejected)
  Rejected because: Evidence shows established adoption of @iarna/toml across 6 files; switching would require migration with no clear benefit demonstrated
  When valid: If @iarna/toml is deprecated or has critical unresolved issues
- Allow multiple TOML parsing libraries based on subsystem preference (rejected)
  Rejected because: Multiple parsers introduce compatibility risk, increase dependency footprint, and create inconsistent parsing behavior across configuration contexts
  When valid: Never recommended; if different parsing behavior is required, address through configuration of the standard library
- Use @ltd/j-toml for TOML 1.0 specification compliance (deferred)
  Rejected because: Current implementation with @iarna/toml is working across all observed contexts; migration cost not justified without specific TOML 1.0 feature requirements
  When valid: If TOML 1.0 specification features become required and @iarna/toml does not support them

## Risks

- Library abandonment or critical security vulnerability in @iarna/toml
  Mitigation: Monitor library maintenance status and security advisories; maintain migration plan to alternative TOML parser; ensure parsing logic is isolated behind clear interfaces to enable library substitution
  Owner: Engineering team
- TOML specification evolution may require parser capabilities not supported by @iarna/toml
  Mitigation: Track TOML specification changes; evaluate parser compliance periodically; design configuration schemas to use well-supported TOML features
  Owner: Engineering team
- Performance bottlenecks if TOML parsing becomes a hot path in configuration-heavy operations
  Mitigation: Profile configuration loading performance; implement caching of parsed configuration where appropriate; consider lazy loading for infrequently accessed configuration
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
- Isolate TOML parsing behind clear interfaces or utility functions to enable future library substitution without widespread code changes; the observed pattern of direct parseTOML calls should be wrapped in project-specific parsing utilities
- Implement consistent error handling for TOML parsing failures that includes file path context and actionable error messages for configuration authors
- When adding new configuration files, follow the established pattern of parse-then-validate: use @iarna/toml for parsing, then apply schema validation separately

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and confirm @iarna/toml is declared as a dependency
- Search the codebase for TOML parsing import statements and verify all use @iarna/toml with no alternative TOML parsers present
- Locate and execute the project's test suite covering configuration loading to verify TOML parsing behavior

Accept when:
- Dependency manifest declares @iarna/toml and no alternative TOML parsing libraries are present
- All TOML parsing import statements reference @iarna/toml exclusively
- Configuration loading tests pass, demonstrating successful TOML parsing across all configuration contexts

## Enforcement

- Verified by: Code review verification that new configuration loading code uses @iarna/toml
- Verified by: Dependency audit in CI pipeline to detect introduction of alternative TOML parsing libraries
- Verified by: Static analysis or linting rules to enforce import patterns for TOML parsing
- Violation handling: Code review rejection if alternative TOML parser is introduced without architectural review
- Violation handling: CI pipeline failure if dependency audit detects multiple TOML parsing libraries
- Violation handling: Refactoring requirement for code that bypasses standard TOML parsing approach
- Exception process: Document specific technical requirement that @iarna/toml cannot satisfy
- Exception process: Propose alternative library with justification in architectural review
- Exception process: Obtain approval from engineering leadership before introducing alternative parser
- Exception process: Update this ADR with exception documentation or supersede with new parsing standard