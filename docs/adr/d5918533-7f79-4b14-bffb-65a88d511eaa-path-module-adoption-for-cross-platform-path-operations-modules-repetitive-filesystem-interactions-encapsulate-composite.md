# path Module Adoption for Cross-Platform Path Operations: Modules Repetitive Filesystem Interactions Encapsulate Composite

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Filesystem operations across agent integrations, MCP routing, configuration loaders, and revert engines require consistent path manipulation.
- Manual path manipulation using raw string concatenation or hardcoded delimiter separators introduces cross-platform incompatibilities across operating systems.
- The codebase establishes repeated adoption of the core path module across 21 files to standardize path resolution, normalization, and segment manipulation.

## Problem Statement

Inconsistent path manipulation techniques and manual path string concatenation create platform-specific path separator bugs and path traversal vulnerabilities across operating system environments. The codebase requires a standardized, runtime-provided module for resolving, joining, and normalizing filesystem paths across all agent definitions, MCP propagation handlers, and file operations.

## Decision

1. SHOULD: Modules with repetitive filesystem interactions SHOULD encapsulate composite path operations within designated internal filesystem utility modules rather than duplicating path parsing logic.

## Policy Block

- SHOULD Modules with repetitive filesystem interactions SHOULD encapsulate composite path operations within designated internal filesystem utility modules rather than duplicating path parsing logic.

In scope:
- All codebase modules performing filesystem path construction, resolution, joining, or normalization.
- Agent adapters, MCP propagation scripts, revert synchronization engines, and CLI handlers manipulating file references.

Out of scope:
- Uniform Resource Identifier manipulation and network protocol endpoints where URL parsing specifications govern formatting.
- Pure in-memory data structures that do not interface with the underlying filesystem.

## Rationale

- Empirical IR analysis confirms 21 files across agents, core libraries, and CLI handlers consistently import and rely on the core path module.
- Standardizing on the core path module guarantees platform-independent path delimiter resolution across varying runtime environments without third-party dependency bloat.
- Centralizing path resolution eliminates subtle platform-specific path parsing failures when coordinating agent configurations across diverse operating systems.

## Consequences

Positive:
- Eliminates cross-platform path delimiter defects across disparate host environments.
- Avoids extraneous external package dependencies for baseline path resolution operations.
- Ensures unified path handling conventions across all agent implementations and core engines.

Negative:
- Requires developers to import and invoke specific path resolution methods instead of utilizing inline string templates.
- Core runtime module behavior remains tied to the active host platform environment unless explicit posix or win32 namespace functions are intentionally designated.

## Alternatives

- Manual string concatenation using slash delimiters (rejected)
  Rejected because: Causes path resolution failures on host environments with differing native delimiter semantics and increases vulnerability to malformed path segments.
  When valid: Only valid for fixed URL path schemas that are strictly platform-agnostic and never touch local disk storage.
- Adopting third-party path manipulation libraries (rejected)
  Rejected because: Adds unnecessary external dependency footprint and maintenance overhead when the core runtime module satisfies all path handling requirements.
  When valid: When complex globbing or advanced pattern matching capabilities beyond basic path operations are mandatory.

## Risks

- Inadvertent use of platform-dependent path methods leading to divergence across development and production environments.
  Mitigation: Encapsulate complex path derivations inside dedicated utility modules and execute automated cross-platform test suites.
  Owner: engineering team
- Regressions caused by direct string manipulations bypassing the path module in newly added agent integrations.
  Mitigation: Enforce static analysis linting rules that disallow raw path separator string splitting and concatenation.
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
- Locate internal path and filesystem utility modules to reuse existing cross-platform path resolution helpers before implementing custom path logic.
- When handling paths intended for cross-system serialization or virtual file hierarchies, evaluate whether POSIX-normalized representations are required before persisting.

## Continuation Context


Verify commands:
- Discover the project verification script from the repository manifest and execute the test suite across supported environments to validate path resolution.
- Inspect the project linter configuration and run the code quality check to verify that no prohibited raw path string concatenations exist.

Accept when:
- All automated test suites pass across all supported host operating systems without path separator failures.
- Static analysis checks complete with zero violations regarding unescaped path delimiters or raw string path concatenations.

## Enforcement

- Verified by: Automated continuous integration pipeline executing unit and integration tests across target platforms.
- Verified by: Static analysis and linting rules prohibiting raw string path concatenations.
- Verified by: Peer code review for all new agent integrations and file-handling modules.
- Violation handling: Pull requests containing unapproved raw path string manipulations or platform-specific separator literals are blocked from merging.
- Violation handling: Identified path handling violations must be refactored to use the path module before approval.
- Exception process: Submit an architectural review request detailing why standard path module functions cannot fulfill the specific path manipulation requirement.
- Exception process: Exceptions require approval from the principal engineering team and must be documented with explicit rationale in the consuming module.