# @iarna/toml Library Adoption for TOML Configuration Parsing: Modules That Parse Toml Formatted Documents

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Multiple subsystems within the codebase require parsing and interpreting structured TOML configuration data, including core runtime configuration, agent specifications, and protocol propagation manifests.
- Inconsistent parsing across disparate modules risks subtle incompatibilities in TOML syntax interpretation, data type conversion, and error handling.
- Static analysis indicates standardized usage of @iarna/toml for TOML parsing across configuration loaders, agent definitions, and protocol synchronization routines.

## Problem Statement

Without a standardized TOML parsing library, divergent parsing mechanisms and disparate error-handling semantics across configuration loaders, agent adapters, and protocol handlers lead to inconsistent configuration evaluation, serialization bugs, and maintenance overhead.

## Decision

1. MUST: Modules that parse TOML-formatted documents or configuration payloads MUST use the @iarna/toml library for deserialization.

## Policy Block

- MUST Modules that parse TOML-formatted documents or configuration payloads MUST use the @iarna/toml library for deserialization.

In scope:
- All application modules, configuration loaders, agent drivers, and protocol synchronization components that read or parse TOML data.

Out of scope:
- Modules that process JSON or YAML data streams exclusively without interacting with TOML documents.

## Rationale

- Standardizing on @iarna/toml across all configuration and protocol ingestion paths guarantees uniform TOML parsing semantics and predictable type mapping across subsystems.
- Centralizing on a single TOML parser reduces cognitive overhead and prevents duplicate dependency bloat across core application components.
- Static analysis confirms repeated, consistent invocation of parseTOML across core configuration, agent management, and integration layers.

## Consequences

Positive:
- Uniform parsing behavior and syntax adherence across all TOML configuration files, manifests, and agent specifications.
- Elimination of conflicting TOML parsing dependencies and divergent edge-case handling across independent modules.
- Seamless integration between deserialized configuration payloads and downstream validation logic.

Negative:
- Subsystems become coupled to the operational characteristics and error formats of @iarna/toml.
- Future migrations away from @iarna/toml would require updating parse calls across configuration loaders, agents, and protocol modules.

## Alternatives

- Adopting alternative TOML parsing libraries or utilizing separate TOML engines per module (rejected)
  Rejected because: Introducing multiple parsers causes behavioral discrepancies in parsing corner cases, increases bundle size, and complicates schema validation.
  When valid: Valid only if a subsystem requires streaming parse capabilities or memory constraints not supported by the primary parser.
- Authoring custom internal regular expression or string-splitting TOML parsers (rejected)
  Rejected because: Custom parsing implementations fail to reliably support TOML specification compliance, multi-line values, inline tables, and escape sequences.
  When valid: Valid only in minimal bootstrap environments where external dependencies are strictly prohibited.

## Risks

- Changes in upstream TOML specification support or breaking changes across major library releases could affect configuration ingestion.
  Mitigation: Lock exact dependency versions in repository lock artifacts and validate parsing logic through automated unit and integration suites.
  Owner: Engineering Team
- Malformed TOML input could cause runtime exceptions if parsing failures are not cleanly caught and translated into actionable configuration errors.
  Mitigation: Encapsulate TOML parsing within error-handling boundaries and validate all parsed structures against declarative schemas.
  Owner: Engineering Team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Verify that all calls to TOML parsing functions cleanly handle parsing exceptions and bubble descriptive syntax errors with line and column information when available.
- Couple TOML parsing directly with type-safe schema validation to ensure deserialized data matches expected runtime models.

## Continuation Context


Verify commands:
- Discover and execute the project automated test runner to run unit and integration tests covering TOML configuration parsing.
- Discover and execute the project linter and type checker to verify that all TOML parsing calls strictly adhere to module type definitions.

Accept when:
- All automated test suites covering configuration loading and TOML parsing pass without errors or regressions.
- Static type analysis and linting checks complete with zero errors across all modules importing the TOML parser.

## Enforcement

- Verified by: Automated continuous integration test suites verifying configuration parsing behavior.
- Verified by: Static analysis and peer code reviews ensuring no unapproved parsing libraries or custom parsers are introduced.
- Violation handling: Pull requests introducing alternative TOML parsers or unhandled parse calls will be blocked by review and required to adopt the standard parser.
- Exception process: Exceptions require submission of an architectural change proposal documenting specific technical requirements that the standard parser cannot fulfill, approved by technical leadership.