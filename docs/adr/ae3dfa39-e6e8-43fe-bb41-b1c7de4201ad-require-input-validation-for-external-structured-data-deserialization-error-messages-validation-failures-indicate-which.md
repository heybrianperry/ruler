# Require Input Validation for External Structured Data Deserialization: Error Messages Validation Failures Indicate Which

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The codebase deserializes external structured data (JSON, TOML) from file system sources across agent implementations, MCP propagation modules, CLI handlers, and path utilities
- Pattern analysis detected direct deserialization calls without explicit validation or error handling in 9 files with 0.93 significance
- Agent configuration loading, MCP server setup, and CLI tooling depend on parsing external files to initialize runtime state
- Unvalidated deserialization introduces security and reliability risks when external files are malformed, corrupted, or maliciously crafted

## Problem Statement

External structured data deserialization occurs at multiple module boundaries without explicit validation, schema enforcement, or error handling. This creates security vulnerabilities (injection attacks, denial of service through malformed input) and reliability issues (runtime crashes from unexpected data shapes). The architecture requires a consistent approach to validating external input before it influences runtime behavior.

## Decision

1. SHOULD: Error messages from validation failures SHOULD indicate which field or constraint failed and provide guidance for correction

## Policy Block

- SHOULD Error messages from validation failures SHOULD indicate which field or constraint failed and provide guidance for correction

In scope:
- All agent configuration loading from external files
- All MCP server definition parsing and propagation
- All CLI command handlers that read structured data from the file system
- All utility modules that deserialize external JSON or TOML content
- Any module boundary where external structured data enters the application runtime

Out of scope:
- Deserialization of data from trusted internal sources that are generated and validated by the application itself
- Parsing of hardcoded or embedded structured data within source code
- Test fixtures where validation is explicitly bypassed for testing error handling paths

## Rationale

- Pattern detection identified direct deserialization without validation across 9 files in agent, MCP, CLI, and utility modules, establishing this as a cross-cutting architectural concern
- Unvalidated external input is a primary attack vector for injection vulnerabilities and a common source of runtime failures in production systems
- Explicit validation at deserialization boundaries creates a clear security perimeter and improves error diagnostics by failing fast with actionable messages
- Schema-based validation enables the codebase to evolve configuration formats safely by detecting breaking changes at load time rather than during execution

## Consequences

Positive:
- Reduces security attack surface by rejecting malformed or malicious input at module boundaries before it influences runtime behavior
- Improves reliability by failing fast with clear error messages when external files are corrupted or have unexpected structure
- Enables safer configuration format evolution by detecting schema mismatches at load time
- Provides better developer experience through actionable validation error messages that indicate exactly what is wrong with the input

Negative:
- Increases initial implementation effort to define validation schemas and integrate validation logic at all deserialization points
- Adds runtime overhead for validation operations, though this is typically negligible for configuration loading scenarios
- Requires ongoing maintenance to keep validation schemas synchronized with evolving data formats
- May introduce breaking changes if existing external files do not conform to newly enforced schemas

## Alternatives

- Continue direct deserialization without validation, relying on runtime type checking and error handling in consuming code (rejected)
  Rejected because: Pushes validation responsibility to every consumer, creating inconsistent error handling and leaving security vulnerabilities at module boundaries. Pattern detection shows this approach is already causing architectural debt across 9 files.
  When valid: Only acceptable for internal data generated and validated by the application itself, not external input
- Implement validation only for security-critical paths (MCP server configs, agent definitions) but allow direct parsing for low-risk utilities (rejected)
  Rejected because: Creates inconsistent security posture and makes it difficult to determine which paths are truly low-risk. Configuration files often become security-critical as systems evolve.
  When valid: Could be considered as a phased rollout strategy, starting with highest-risk modules first
- Use runtime type checking without explicit validation schemas, relying on language type system to catch errors (rejected)
  Rejected because: Provides weaker guarantees than schema validation, offers poor error messages, and does not protect against malicious input that is well-typed but semantically invalid
  When valid: May be sufficient for internal APIs with strong type contracts, but inadequate for external file input

## Risks

- Existing external configuration files may not conform to newly enforced validation schemas, causing breaking changes during rollout
  Mitigation: Audit existing configuration files against proposed schemas before enforcement. Provide migration tooling or clear upgrade documentation. Consider a grace period with warnings before hard failures.
  Owner: Engineering team
- Validation schema maintenance may lag behind data format evolution, causing false positives or allowing invalid data through outdated schemas
  Mitigation: Colocate validation schemas with consuming modules to increase visibility. Include schema updates in code review process for any configuration format changes. Add tests that verify schema coverage.
  Owner: Engineering team
- Overly strict validation may reject valid edge cases or future-compatible extensions, reducing flexibility
  Mitigation: Design schemas to allow optional fields and unknown properties where appropriate. Document extension points explicitly. Use SHOULD-level rules for recommendations rather than MUST-level for all constraints.
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
- Define validation schemas as typed contracts that can be reused across modules. Consider using schema validation libraries that provide both runtime validation and static type inference.
- Wrap all deserialization operations in try-catch blocks that distinguish between parse errors (malformed syntax) and validation errors (invalid structure). Provide different error messages for each case.
- For agent configuration and MCP server definitions, consider implementing a two-phase load: parse to detect syntax errors, then validate to detect schema violations. This provides clearer error diagnostics.
- Log validation failures with sufficient context (file path, failed constraint, actual vs expected value) to enable rapid debugging without exposing sensitive data in error messages.

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and identify any validation or schema libraries already present in the dependency graph
- Search the codebase for deserialization operations and verify each is wrapped in error handling and preceded by validation logic
- Locate the project's test suite and verify that validation error paths are covered by tests that assert correct error messages and rejection behavior

Accept when:
- All deserialization operations in agent, MCP, CLI, and utility modules are wrapped in error handling that catches and logs parse failures
- Validation schemas are defined for all external structured data formats and validation occurs before deserialized data reaches application logic
- Tests demonstrate that malformed input and schema violations are rejected with actionable error messages

## Enforcement

- Verified by: Code review process verifies that all new deserialization operations include validation and error handling
- Verified by: Static analysis or linting rules detect direct deserialization calls without surrounding validation logic
- Verified by: Security review process audits module boundaries for unvalidated external input
- Violation handling: Code review blocks merge of changes that introduce unvalidated deserialization at module boundaries
- Violation handling: Static analysis failures in continuous integration prevent deployment of code with validation gaps
- Violation handling: Security incidents involving malformed input trigger immediate audit of affected modules and remediation
- Exception process: Exception requests must document why validation is not required for the specific deserialization operation
- Exception process: Security team must approve exceptions for any external input deserialization
- Exception process: Approved exceptions must be documented in code comments with rationale and expiration date for review