# Use of JSON.parse() and TOML Parsing for Secure Input Validation: Implement Robust Error Handling Mechanisms Around

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- External configuration and data files are frequently used across various components, including agents and CLI tools.
- These files contain structured data (JSON, TOML) that needs to be deserialized into program objects.
- Ensuring the integrity and correct format of this input is critical for application stability and security.
- The codebase consistently uses specific parsing functions for these data formats.

## Problem Statement

The system requires a consistent and secure method for deserializing structured data from external sources, particularly JSON and TOML files, to prevent issues arising from malformed or malicious input.

## Decision

1. SHOULD: SHOULD implement robust error handling mechanisms around all parsing operations to gracefully manage malformed or invalid input.

## Policy Block

- SHOULD SHOULD implement robust error handling mechanisms around all parsing operations to gracefully manage malformed or invalid input.

In scope:
- Modules responsible for loading configuration files.
- Agent implementations that process external data.
- CLI commands that consume structured input from files.

Out of scope:
- Internal data structures that are not exposed to external modification.
- Parsing of unstructured text data.

## Rationale

- JSON.parse() and parseTOML() provide efficient and standard mechanisms for deserializing common structured data formats.
- Their consistent application across the codebase ensures a uniform approach to input processing.
- These functions inherently perform basic structural validation, rejecting malformed input before it can cause runtime errors or security vulnerabilities.
- The observed usage indicates a deliberate choice to rely on these parsers for initial input integrity.

## Consequences

Positive:
- Consistent and predictable deserialization of structured data.
- Basic structural validation of external input, reducing the risk of runtime errors from malformed files.
- Improved code readability and maintainability due to standardized parsing methods.

Negative:
- Reliance on the inherent security of the chosen parsers; deeper semantic validation may require additional mechanisms.
- Potential for parser-specific vulnerabilities if not kept up-to-date or used carefully.
- Error messages from raw parsing might not always be user-friendly without additional wrapping.

## Alternatives

- Manual parsing or custom parsers. (rejected)
  Rejected because: Increases development effort, introduces potential for custom parsing bugs, and deviates from established library/language features.
  When valid: For highly specialized or proprietary data formats not supported by existing libraries.
- Using schema validation libraries (e.g., JSON Schema, Zod). (deferred)
  Rejected because: While offering more robust validation, the current pattern focuses on basic deserialization and structural integrity.
  When valid: When strict data contracts and advanced semantic validation are required beyond basic structural parsing.

## Risks

- Maliciously crafted input could exploit parser vulnerabilities.
  Mitigation: Keep parsing libraries updated, implement additional input sanitization and validation layers where sensitive data is involved.
  Owner: Engineering team.
- Performance overhead for very large input files.
  Mitigation: Implement streaming parsers or process data in chunks for extremely large inputs.
  Owner: Engineering team.

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Consider wrapping parsing operations in utility functions to centralize error handling and provide more informative messages.
- For critical configurations, implement additional validation steps beyond basic parsing, such as schema validation.

## Continuation Context


Verify commands:
- Inspect the project's dependency manifest to identify parsing libraries.
- Examine relevant agent, CLI, and core configuration files for usage of JSON.parse() and parseTOML().
- Run unit and integration tests that cover configuration loading and external data processing.

Accept when:
- All external JSON and TOML data is successfully parsed without runtime errors.
- Malformed input consistently results in controlled error handling, not application crashes.
- No direct usage of alternative parsing methods for JSON or TOML is found in new code.

## Enforcement

- Verified by: Automated CI checks for adherence to parsing function usage.
- Verified by: Code reviews ensuring proper error handling and consistent application of parsing methods.
- Violation handling: CI pipeline failures for non-compliant code.
- Violation handling: Code review comments requiring remediation.
- Exception process: Exceptions require explicit approval from a lead architect or security officer, documented with a clear rationale for deviation.