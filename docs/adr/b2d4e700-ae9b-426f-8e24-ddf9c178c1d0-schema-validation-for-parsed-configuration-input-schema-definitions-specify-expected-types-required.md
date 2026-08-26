# Schema Validation for Parsed Configuration Input: Schema Definitions Specify Expected Types Required

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The codebase processes external configuration files in JSON and TOML formats across agent modules, MCP integration, CLI handlers, and core configuration loading
- Configuration parsing occurs in 9 files using native JSON parsing and TOML parsing libraries to deserialize file content into runtime objects
- The core configuration loader implements schema validation using a validation library, defining explicit schemas for configuration structure, types, and constraints
- Agent modules and MCP propagation modules perform direct parsing of configuration files without consistent schema validation, creating inconsistent input validation coverage
- The security.input_validation facet indicates this pattern addresses the architectural concern of validating untrusted external input before use in application logic

## Problem Statement

External configuration files represent untrusted input that can contain malformed data, unexpected types, or malicious payloads. Direct parsing without validation exposes the application to runtime errors, type confusion, and potential security vulnerabilities. The codebase exhibits inconsistent validation practices: the core configuration loader validates input through schemas, while agent and MCP modules parse configuration directly without validation. This inconsistency creates gaps in input validation coverage and increases the risk of processing invalid or malicious configuration data.

## Decision

1. MUST: Schema definitions MUST specify expected types, required fields, optional fields, and valid value constraints for all configuration properties

## Policy Block

- MUST Schema definitions MUST specify expected types, required fields, optional fields, and valid value constraints for all configuration properties

In scope:
- All modules that read and parse configuration files from the file system
- Agent implementations that load agent-specific configuration
- MCP integration modules that process MCP server definitions
- CLI command handlers that read project metadata or configuration
- Core configuration loading infrastructure
- Any module that deserializes external JSON or TOML data into runtime objects

Out of scope:
- Hardcoded configuration constants defined in source code
- Internal data structures passed between trusted modules within the application
- Runtime-generated configuration objects that do not originate from external files
- Test fixtures with known-valid structure used in unit tests

## Rationale

- The evidence shows 9 files performing configuration parsing, with only the core configuration loader implementing schema validation, indicating an architectural gap in input validation coverage
- Schema validation provides type safety, constraint enforcement, and explicit error handling at the boundary between external input and application logic, preventing invalid data from propagating through the system
- The presence of comprehensive schema definitions in ConfigLoader.ts demonstrates the project's architectural intent to validate configuration input, though implementation is incomplete across other modules
- Consistent schema validation across all configuration parsing points reduces the attack surface for malformed or malicious configuration files and improves error diagnostics

## Consequences

Positive:
- Type safety guarantees that configuration objects match expected structure before use, eliminating runtime type errors from malformed configuration
- Explicit validation failures provide clear error messages indicating which configuration constraint was violated, improving debugging and user experience
- Schema definitions serve as executable documentation of configuration structure and constraints
- Validation at input boundaries prevents invalid data from propagating through the application, reducing defensive checks in downstream logic

Negative:
- Schema definitions add code overhead and must be maintained alongside configuration structure changes
- Validation adds runtime overhead to configuration loading, though this is typically negligible for startup-time configuration
- Overly strict schemas may reject valid configuration variations, requiring schema updates to accommodate legitimate use cases
- Developers must learn the schema validation library's API and patterns to implement validation correctly

## Alternatives

- Direct parsing without validation, relying on runtime errors to surface configuration problems (rejected)
  Rejected because: Runtime errors from invalid configuration provide poor error messages, occur deep in application logic rather than at input boundaries, and expose the application to type confusion vulnerabilities
  When valid: Only acceptable for internal test fixtures with known-valid structure
- Manual validation using conditional checks and type guards after parsing (rejected)
  Rejected because: Manual validation is error-prone, verbose, difficult to maintain, and lacks the composability and type inference benefits of schema validation libraries
  When valid: May be appropriate for simple single-field validations where schema library overhead is unjustified
- TypeScript interface definitions without runtime validation (rejected)
  Rejected because: TypeScript types are erased at runtime and provide no protection against invalid external input, only compile-time type checking for code that assumes valid input
  When valid: Appropriate for internal type safety after validation has occurred at input boundaries

## Risks

- Incomplete migration: existing modules continue parsing without validation, leaving validation gaps
  Mitigation: Audit all configuration parsing sites, add validation to modules lacking it, and establish verification commands to detect unvalidated parsing
  Owner: engineering team
- Schema drift: schemas become outdated as configuration structure evolves, causing false validation failures
  Mitigation: Colocate schemas with consuming modules, include schema updates in configuration change reviews, and maintain test coverage for schema validation
  Owner: engineering team
- Overly permissive schemas: schemas accept invalid configurations by using loose types or missing constraints
  Mitigation: Define schemas with strict types and explicit constraints, use strict mode where available, and review schemas for completeness during code review
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
- Examine the existing schema definitions in the core configuration loader to understand the project's schema validation patterns, including how schemas define required vs optional fields, type constraints, nested objects, and error handling
- When adding validation to existing parsing code, wrap the parsing and validation in a try-catch block that catches both parse errors and validation errors, providing context about which file and operation failed
- For agent modules that load agent-specific configuration, define schemas that validate only the configuration properties relevant to that agent, allowing the schema to evolve independently of other agent configurations

## Continuation Context


Verify commands:
- Discover the project's static analysis or linting configuration and execute the verification script to detect direct parsing operations without subsequent validation
- Locate the project's test suite and execute tests that verify schema validation behavior, including tests for valid configuration acceptance and invalid configuration rejection
- Search the codebase for parsing function invocations and verify each is followed by schema validation before the parsed data is used in application logic

Accept when:
- All configuration parsing sites identified in the codebase include schema validation before parsed data is used
- Schema validation tests pass, demonstrating that valid configurations are accepted and invalid configurations are rejected with clear error messages
- Static analysis or code review confirms no direct parsing of external configuration without validation

## Enforcement

- Verified by: Code review verifying that all configuration parsing includes schema validation
- Verified by: Static analysis or linting rules detecting unvalidated parsing operations
- Verified by: Test coverage requirements for schema validation paths including both valid and invalid input cases
- Violation handling: Code review rejects changes that parse external configuration without schema validation
- Violation handling: Static analysis failures block merge until validation is added
- Violation handling: Runtime validation failures log errors and prevent application startup with invalid configuration
- Exception process: Exceptions require explicit justification in code review explaining why validation is not applicable
- Exception process: Exception approval requires demonstration that the input source is trusted and cannot contain invalid data
- Exception process: Approved exceptions must be documented with inline comments explaining the validation exemption rationale