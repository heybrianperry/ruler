# Validate Untrusted Input from Configuration Files Before Parsing: Schema Validation Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all code that reads and parses external configuration files.

## Context

- The codebase reads configuration from external files (VSCode settings.json and OpenHands TOML) that may contain malformed or malicious content
- Both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts parse untrusted input using JSON.parse() and parseTOML() without validation
- Configuration data is subsequently stored in Map-based cache layers (existingServerMap, existingSseServers, existingStdioServers) and used to control server behavior
- The fs and fs/promises modules are used to read file content directly from disk, exposing the system to file-based injection vectors
- Public API contracts (VSCodeSettings, AugmentMcpServer, readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands) expose parsed configuration to downstream consumers

## Problem Statement

Configuration files read from disk are parsed directly without schema validation or sanitization, creating a security vulnerability where malformed or malicious input can cause runtime errors, data corruption in cache layers, or exploitation of downstream systems that trust the parsed configuration data.

## Decision

1. SHOULD: Schema validation SHOULD use a validation library (e.g., zod, joi, ajv) rather than manual field checking

## Policy Block

- SHOULD Schema validation SHOULD use a validation library (e.g., zod, joi, ajv) rather than manual field checking

In scope:
- All functions that read configuration files using fs or fs/promises modules
- All parsing operations using JSON.parse(), parseTOML(), or similar deserialization functions
- All code paths that store parsed configuration in Map-based cache layers
- Public API functions that expose configuration data (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands)

Out of scope:
- Configuration data that originates from trusted internal sources (hardcoded defaults, environment variables controlled by deployment)
- Parsing of data from authenticated API responses with TLS verification
- Test fixtures and mock data used exclusively in unit tests

Exceptions:
- EXC-001: Configuration files are generated and controlled entirely by the application itself with no external modification possible

## Rationale

- The evidence shows direct parsing of file content without validation in two critical configuration paths (VSCode settings and OpenHands MCP propagation)
- Unvalidated configuration data flows into Map-based cache layers that control server behavior, creating a trust boundary violation
- The public API contracts expose these parsing functions to external callers, amplifying the impact of any parsing vulnerabilities
- Schema validation at the input boundary prevents malformed data from propagating through the system and enables fail-fast behavior with clear error messages

## Consequences

Positive:
- Prevents runtime crashes and undefined behavior caused by malformed configuration files
- Establishes a clear trust boundary between external file input and internal data structures
- Enables early detection of configuration errors with actionable error messages for users
- Reduces attack surface by rejecting malicious input before it reaches business logic or cache layers

Negative:
- Adds parsing overhead and latency to configuration loading operations
- Requires maintenance of schema definitions that must evolve with configuration format changes
- May reject previously accepted but technically invalid configuration files, breaking backward compatibility
- Increases code complexity with validation logic and error handling paths

## Alternatives

- Continue parsing configuration files without validation, relying on downstream error handling (rejected)
  Rejected because: Downstream error handling cannot distinguish between malformed input and logic errors, and allows invalid data to corrupt cache layers before detection
  When valid: Never valid for untrusted external input
- Implement manual field-by-field validation without a schema validation library (rejected)
  Rejected because: Manual validation is error-prone, difficult to maintain, and lacks the composability and type safety of schema validation libraries
  When valid: Only for extremely simple configuration with 1-2 fields where library overhead is unjustified
- Use TypeScript type guards with runtime type checking libraries (zod, io-ts) for validation (accepted)
  When valid: Preferred approach that provides both compile-time type safety and runtime validation

## Risks

- Schema validation may be bypassed if new configuration loading code paths are added without following the validation pattern
  Mitigation: Implement linting rules or static analysis to detect JSON.parse() and parseTOML() calls without surrounding validation, enforce through CI checks
  Owner: Engineering team
- Overly strict validation schemas may reject valid edge cases, causing operational issues for users with unusual but legitimate configurations
  Mitigation: Design schemas to be permissive for optional fields, provide clear validation error messages, and maintain a process for schema evolution based on user feedback
  Owner: Engineering team and product team
- Performance degradation on large configuration files if validation is not optimized
  Mitigation: Profile validation performance, implement streaming validation for large files if needed, cache validated configuration to avoid repeated validation
  Owner: Engineering team

## Implementation Notes

- Add zod or joi as a dependency and define schemas for VSCodeSettings and OpenHands TOML configuration structures
- Wrap JSON.parse() and parseTOML() calls in validation functions that return Result<T, ValidationError> types
- Update readVSCodeSettings and propagateMcpToOpenHands to handle validation failures and return meaningful error messages
- Add unit tests that verify rejection of malformed configuration files with various invalid inputs (missing fields, wrong types, malicious content)
- Document the configuration schema in user-facing documentation so users understand validation requirements

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' src/ | grep -v 'validate\|schema\|parse.*catch' || echo 'All JSON.parse calls are validated'
- grep -r 'parseTOML' src/ | grep -v 'validate\|schema\|parse.*catch' || echo 'All parseTOML calls are validated'
- npm test -- --grep 'configuration.*validation' && echo 'Configuration validation tests pass'

Accept when:
- All grep commands return no unvalidated parsing calls or echo success messages
- Configuration validation test suite passes with 100% coverage of validation logic
- Manual testing confirms that malformed configuration files are rejected with clear error messages without crashing the application

## Enforcement

- Verified by: Static analysis in CI pipeline to detect unvalidated JSON.parse() and parseTOML() calls
- Verified by: Code review checklist requiring validation for all new configuration parsing code
- Verified by: Unit test coverage requirements for configuration validation paths
- Violation handling: CI build fails if static analysis detects unvalidated parsing operations
- Violation handling: Pull requests are blocked until validation is added and tests demonstrate rejection of invalid input
- Violation handling: Security team is notified of violations in production code for incident response
- Exception process: Developer submits exception request with justification to security team
- Exception process: Security team reviews the trust boundary and determines if input is truly trusted
- Exception process: If approved, exception is documented in code comments with ticket reference and expiration date
- Exception process: Exceptions are reviewed quarterly and must be re-justified or remediated