# Validate External Data with Parsing Functions Before Access Pattern Queries: Data Access Patterns

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Test suites for MistralVibeAgent and apply-engine load configuration data from external sources (TOML files, JSON files) and immediately query the parsed structures using array find operations
- The codebase uses parseTOML and JSON.parse to transform external file content into structured data before applying data access patterns like find() queries
- Configuration files contain MCP server definitions and ruler configurations that must be validated during parsing to prevent runtime errors in subsequent access operations
- The pattern appears in unit test contexts where fs/promises reads file content, parsing functions validate structure, and find() operations locate specific configuration entries

## Problem Statement

External configuration data loaded from TOML and JSON files requires validation before data access patterns are applied. Without parsing-time validation, malformed or unexpected data structures can cause runtime errors, test failures, or security vulnerabilities when find() and other query operations assume specific object shapes and properties.

## Decision

1. MUST: Data access patterns (find, filter) applied to parsed external data MUST handle cases where expected entries are not found

## Policy Block

- MUST Data access patterns (find, filter) applied to parsed external data MUST handle cases where expected entries are not found

## Rationale

- Evidence shows parseTOML(content) and JSON.parse(await fs.readFile(...)) are consistently applied before find() operations in test files, establishing a validation-before-access pattern
- The pattern prevents runtime type errors by ensuring external data conforms to expected structure before queries assume specific object shapes
- Separating parsing from access enables early failure detection and clearer error messages when configuration files are malformed
- The pattern appears in 2 test files with 91.25% confidence, indicating consistent architectural practice for configuration handling

## Consequences

Positive:
- Parsing functions catch malformed data early, before access patterns execute, reducing runtime errors
- Clear separation between validation (parsing) and querying (find/filter) improves code maintainability and testability
- Type safety improves as parsing functions can enforce schema constraints before data access operations
- Test suites can verify configuration loading independently from application runtime behavior

Negative:
- Additional parsing step adds latency to configuration loading, though typically negligible for file-based configs
- Parsing functions may silently coerce or transform data in unexpected ways if not carefully implemented
- Error handling becomes more complex as failures can occur at both parsing and access stages

## Alternatives

- Direct access to file content without parsing validation (rejected)
  Rejected because: Skipping validation allows malformed data to reach access patterns, causing runtime errors with unclear root causes
  When valid: Never valid for external configuration data; only acceptable for trusted in-memory structures
- Lazy validation during first access operation (rejected)
  Rejected because: Delays error detection and couples validation logic with every access point, violating separation of concerns
  When valid: May be valid for performance-critical paths where upfront parsing cost is prohibitive
- Schema validation libraries (Zod, Joi) instead of basic parsing (deferred)
  Rejected because: Not rejected; represents enhancement to current pattern with stronger type guarantees
  When valid: Valid when configuration complexity justifies additional dependency and schema maintenance overhead

## Risks

- Parsing functions may have vulnerabilities (prototype pollution, injection) that compromise validation
  Mitigation: Use well-maintained parsing libraries (@iarna/toml, native JSON.parse) and keep dependencies updated; consider schema validation for additional safety
  Owner: engineering team
- Inconsistent application of pattern across codebase may leave some configuration loading paths unvalidated
  Mitigation: Establish linting rules or code review checklist to verify parsing precedes access; document pattern in architecture guidelines
  Owner: engineering team
- Parsing errors may not provide sufficient context for debugging malformed configuration files
  Mitigation: Wrap parsing calls with try-catch blocks that enrich errors with file path and content context
  Owner: engineering team

## Implementation Notes

- When loading TOML configuration, use parseTOML(content) immediately after fs.readFile and before any find() or property access
- When loading JSON configuration, use JSON.parse(await fs.readFile(path, 'utf8')) as atomic operation before querying parsed structure
- Wrap parsing operations in try-catch blocks to handle malformed files gracefully and provide clear error messages with file context
- Consider adding TypeScript type assertions or runtime schema validation after parsing to enforce expected structure before access patterns

## Continuation Context


Verify commands:
- grep -r 'parseTOML\|JSON\.parse' tests/ --include='*.ts' | grep -v 'find\|filter' || echo 'Parsing without access detected'
- grep -B5 '\.find(' tests/ --include='*.ts' | grep -E 'parseTOML|JSON\.parse' || echo 'Access without parsing detected'
- npm test -- --testPathPattern='MistralVibeAgent|apply-engine' --verbose

Accept when:
- All configuration file reads are followed by parsing function calls before any data access operations
- Test suites for MistralVibeAgent and apply-engine successfully load and query configuration without runtime type errors
- Grep verification confirms parsing functions precede find/filter operations in configuration loading code paths

## Enforcement

- Verified by: Code review checklist verifying parsing precedes access in configuration loading code
- Verified by: Unit test coverage for configuration loading paths with malformed input cases
- Verified by: Static analysis or linting rules detecting direct file content access without parsing
- Violation handling: Code review blocks merge if configuration loading bypasses parsing validation
- Violation handling: CI pipeline fails if tests demonstrate runtime errors from unvalidated configuration access
- Violation handling: Architecture review required for any new configuration loading patterns that deviate from parse-then-access flow
- Exception process: Document exception rationale in code comments explaining why parsing is skipped or deferred
- Exception process: Obtain architecture team approval for alternative validation approaches
- Exception process: Add compensating controls such as runtime type guards or schema validation at access points