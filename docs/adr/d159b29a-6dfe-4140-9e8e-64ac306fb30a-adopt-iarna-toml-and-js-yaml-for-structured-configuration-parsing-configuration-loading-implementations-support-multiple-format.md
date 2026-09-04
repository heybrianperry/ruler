# Adopt @iarna/toml and js-yaml for Structured Configuration Parsing: Configuration Loading Implementations Support Multiple Format

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The system requires loading configuration data from human-editable structured files across multiple components including agent definitions, subagent processors, and unified configuration loaders
- Configuration sources include external files in multiple formats (TOML, YAML, JSON) and environment variables, requiring format-specific parsing capabilities beyond JSON.parse
- TOML and YAML formats provide richer syntax for configuration files compared to JSON, including comments, multi-line strings, and more readable nested structures
- The core infrastructure layer (src/core) and agent subsystem (src/agents) both require consistent structured data parsing capabilities for configuration ingestion

## Problem Statement

The system needs to load and parse configuration data from multiple structured file formats (TOML, YAML, JSON) across agent definitions, subagent processing, and configuration loading subsystems. Native JavaScript provides JSON parsing but lacks built-in support for TOML and YAML formats commonly used for human-editable configuration files.

## Decision

1. SHOULD: Configuration loading implementations SHOULD support multiple format detection and route to the appropriate parser based on file extension or content inspection

## Policy Block

- SHOULD Configuration loading implementations SHOULD support multiple format detection and route to the appropriate parser based on file extension or content inspection

In scope:
- Configuration loading subsystems that read TOML or YAML files
- Agent definition loaders that parse structured configuration files
- Subagent processors that ingest configuration from external files
- Any component in src/core or src/agents that requires structured data parsing from files

Out of scope:
- Runtime data serialization and deserialization (use native JSON)
- Network protocol message parsing
- Binary data parsing
- Components that only consume configuration via dependency injection and do not directly parse files

## Rationale

- Evidence shows @iarna/toml adopted across 3 files (MistralVibeAgent.ts, SubagentsProcessor.ts, UnifiedConfigLoader.ts) with significance 0.89-0.93, indicating established architectural pattern
- TOML and YAML formats enable human-readable configuration with features like comments and multi-line strings that improve maintainability of configuration files
- Multiple format support (TOML, YAML, JSON) provides flexibility for different configuration use cases while maintaining consistent parsing infrastructure
- Centralized parsing library adoption in core infrastructure ensures consistent behavior and error handling across all configuration loading paths

## Consequences

Positive:
- Human-editable configuration files with rich syntax support (comments, multi-line strings, readable nesting)
- Consistent parsing behavior across all configuration loading subsystems
- Reduced implementation complexity by delegating format-specific parsing to specialized libraries
- Multiple format support enables flexibility for different configuration sources and use cases

Negative:
- Additional runtime dependencies increase bundle size and dependency maintenance burden
- Multiple parsing libraries create multiple potential points of failure and security vulnerabilities
- Developers must learn and maintain knowledge of multiple configuration format specifications
- Version incompatibilities or breaking changes in parsing libraries require coordinated updates across multiple subsystems

## Alternatives

- Use only JSON for all configuration files with native JSON.parse (rejected)
  Rejected because: JSON lacks comments and has less readable syntax for complex nested configuration, reducing maintainability of human-edited configuration files
  When valid: Valid for machine-generated configuration or when configuration complexity is minimal
- Implement custom TOML/YAML parsers (rejected)
  Rejected because: Parsing complex structured formats requires significant implementation effort and introduces risk of specification non-compliance and parsing bugs
  When valid: Valid only if third-party libraries have unacceptable security vulnerabilities or licensing issues
- Standardize on single configuration format (TOML only or YAML only) (rejected)
  Rejected because: Evidence shows both formats in active use across different subsystems, suggesting different formats serve different use cases
  When valid: Valid for new projects or during major refactoring when format consolidation is feasible

## Risks

- Parsing library vulnerabilities could expose the system to malicious configuration file attacks
  Mitigation: Monitor security advisories for @iarna/toml and js-yaml, apply updates promptly, validate parsed data structure before use
  Owner: engineering team
- Breaking changes in parsing library APIs could require extensive refactoring across multiple subsystems
  Mitigation: Pin dependency versions in lock file, test parsing behavior in CI, isolate parsing logic behind abstraction layer
  Owner: engineering team
- Inconsistent error handling across different parsers could lead to confusing failure modes
  Mitigation: Implement unified error handling wrapper that normalizes parse errors from different libraries
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
- Wrap parsing operations in try-catch blocks and provide context-rich error messages that include file path, format type, and line number if available from parser
- Consider implementing a configuration loader abstraction that encapsulates format detection and parser selection, isolating direct parser dependencies to a single module
- Validate parsed configuration objects against expected schema using type guards or schema validation before passing to consuming components

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and verify that both parsing libraries are declared as dependencies
- Discover the project's lock artifact and confirm exact resolved versions are recorded
- Discover and execute the project's test suite to verify parsing behavior across all configuration loading subsystems

Accept when:
- Dependency manifest declares both parsing libraries and lock artifact records exact resolved versions
- All test suites pass, confirming parsing operations work correctly across configuration loading subsystems
- Code inspection confirms parsing operations follow error handling and validation patterns specified in rules

## Enforcement

- Verified by: Automated dependency scanning in CI pipeline verifies declared parsing libraries are present
- Verified by: Unit tests verify parsing behavior and error handling for each configuration format
- Verified by: Code review checklist includes verification of proper parser usage and error handling
- Violation handling: CI pipeline fails if parsing library dependencies are missing or have security vulnerabilities
- Violation handling: Code review blocks merge if parsing operations lack proper error handling
- Violation handling: Runtime errors from parsing failures are logged with full context for debugging
- Exception process: Exceptions for alternative parsing libraries require architecture review and documentation of technical justification
- Exception process: Temporary exceptions for security vulnerabilities require documented mitigation plan and timeline for resolution
- Exception process: Exception requests must demonstrate that the alternative approach maintains or improves configuration loading reliability