# Adopt Zod for Runtime Validation of Deserialized External Input: Validation Failures Handled Explicitly Error Logging

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The codebase deserializes external file content (JSON, TOML) into runtime objects across agent implementations, CLI handlers, MCP integration, and configuration loading
- ConfigLoader demonstrates Zod schema validation applied to parsed configuration objects, establishing a validation pattern for external input
- Multiple agent modules and integration points parse file content without visible schema validation, creating potential type safety and security gaps
- The project requires runtime type safety guarantees for configuration and state data that cannot be statically verified at compile time

## Problem Statement

External file content deserialized into runtime objects lacks consistent validation, creating risks of runtime type errors, malformed data propagation, and potential security vulnerabilities. While ConfigLoader demonstrates Zod-based validation, other parsing sites (agents, CLI, MCP integration) show raw deserialization without schema enforcement, leading to inconsistent input handling across the codebase.

## Decision

1. MUST: Validation failures MUST be handled explicitly with error logging or user-facing error messages, preventing invalid data from propagating into application state

## Policy Block

- MUST Validation failures MUST be handled explicitly with error logging or user-facing error messages, preventing invalid data from propagating into application state

In scope:
- All file I/O operations that deserialize structured text formats (JSON, TOML, YAML, XML) into runtime objects
- Configuration loading from user-editable files or environment sources
- Agent state persistence and restoration from external storage
- CLI command input parsing beyond simple string arguments
- MCP server configuration and message payload deserialization
- Any external data source where structure and types cannot be statically guaranteed

Out of scope:
- Internal module-to-module data passing where TypeScript types provide compile-time guarantees
- Hardcoded or literal data structures defined in source code
- Binary deserialization formats with their own type systems (Protocol Buffers, MessagePack with schemas)
- Simple string or primitive parsing that does not produce complex objects

## Rationale

- ConfigLoader demonstrates successful Zod adoption with comprehensive schema definitions for configuration validation, establishing a proven pattern within the codebase
- Runtime validation catches malformed input at deserialization boundaries, preventing type errors and invalid state from propagating through the application
- Zod provides TypeScript type inference from schemas, maintaining type safety across the validation boundary without duplicate type definitions
- Consistent validation across all external input points reduces security surface area and improves error diagnostics

## Consequences

Positive:
- Runtime type safety for external input eliminates a class of type errors that TypeScript cannot prevent statically
- Explicit schemas serve as living documentation of expected data structures and constraints
- Validation failures surface early with clear error messages, improving debuggability and user experience
- Type inference from Zod schemas reduces boilerplate and keeps types synchronized with validation logic

Negative:
- Adds runtime validation overhead to all deserialization paths, though typically negligible compared to I/O costs
- Requires schema maintenance alongside data structure evolution, creating an additional change surface
- Increases bundle size and dependency footprint with the Zod library
- May require refactoring existing parsing code that currently lacks validation

## Alternatives

- Use TypeScript type assertions without runtime validation (rejected)
  Rejected because: Type assertions provide no runtime safety and allow malformed external input to violate type contracts, leading to runtime errors and potential security issues
  When valid: Only for internal data structures where compile-time types are sufficient and input source is fully trusted
- Implement custom validation functions for each data structure (rejected)
  Rejected because: Custom validation lacks type inference, requires duplicate type definitions, and creates inconsistent validation patterns across the codebase without the composability and refinement features Zod provides
  When valid: For highly specialized validation logic that cannot be expressed in declarative schema form
- Adopt alternative schema validation libraries (Yup, io-ts, ArkType) (rejected)
  Rejected because: ConfigLoader already demonstrates Zod adoption with extensive schemas; switching would require migration effort without clear architectural benefit, and Zod provides strong TypeScript integration with active maintenance
  When valid: In greenfield projects or when specific library features (e.g., io-ts functional programming style) align better with team preferences

## Risks

- Incomplete adoption leaves validation gaps where some parsing sites lack schema enforcement, creating inconsistent security posture
  Mitigation: Audit all deserialization sites identified in evidence (agents, CLI, MCP modules) and systematically add Zod validation; establish linting or code review checks for new parsing code
  Owner: Engineering team
- Overly permissive schemas (excessive use of optional fields, unknown types, or catchall patterns) may pass invalid data while appearing to validate
  Mitigation: Schema definitions should default to strict mode and explicitly document why fields are optional or why catchall patterns are necessary; code review should scrutinize schema strictness
  Owner: Engineering team
- Schema evolution may break backward compatibility with existing serialized data, causing validation failures on legitimate input
  Mitigation: Use Zod's optional and default value features to maintain backward compatibility; version schemas when breaking changes are necessary; test validation against historical data samples
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
- Reference ConfigLoader's schema definitions as the canonical example of Zod usage patterns within this codebase, including object schemas, enum validation, optional fields, and nested structures
- For agent state deserialization, define schemas that match the persisted JSON structure and validate immediately after parsing, before constructing agent instances or updating state
- When adding validation to existing parsing sites, preserve error handling behavior (e.g., FirebenderAgent's console.warn pattern) while adding schema validation before the existing error boundary

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and identify the schema validation library and its resolved version in the lock artifact
- Discover and execute the project's static analysis or linting configuration to verify Zod schema usage at deserialization boundaries
- Discover and execute the project's test suite, filtering for validation test cases that exercise schema enforcement with both valid and invalid input

Accept when:
- All deserialization sites identified in the evidence (agents, CLI, MCP modules, configuration loader) have Zod schema validation with explicit error handling
- Schema definitions exist for all external data structures with appropriate strictness (required fields, type constraints, enums)
- Test coverage includes validation failure cases demonstrating that malformed input is rejected with clear error messages

## Enforcement

- Verified by: Code review verifies that all new deserialization code includes Zod schema validation before data use
- Verified by: Static analysis or linting rules detect raw JSON.parse or TOML parse calls without subsequent validation
- Verified by: Integration tests exercise validation boundaries with both valid and invalid input samples
- Violation handling: Code review blocks merge of deserialization code lacking schema validation
- Violation handling: Runtime validation failures log errors and prevent invalid data from entering application state
- Violation handling: Security review flags validation gaps during threat modeling or audit processes
- Exception process: Exceptions require explicit justification documenting why the input source is trusted and validation is unnecessary
- Exception process: Exceptions must be approved by a senior engineer or security reviewer
- Exception process: Approved exceptions are documented in code comments with rationale and risk acceptance