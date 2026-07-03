# Validate JSON Input Before Parsing in VSCode Settings Management: Validation Logic Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase reads and writes VSCode settings files using filesystem operations ('fs', 'path') to persist configuration data for MCP server management
- Settings data is stored as JSON and parsed using JSON.parse(content) without explicit validation, creating potential for runtime errors on malformed input
- The module exports public contracts (VSCodeSettings, AugmentMcpServer, readVSCodeSettings, writeVSCodeSettings, transformRulerToAugmentMcp) that handle server configuration state
- A cache layer pattern (existingServerMap.set) is used to maintain in-memory representations of server configurations after parsing
- The settings management operates in a concurrent environment where multiple read/write operations may occur

## Problem Statement

Direct JSON.parse() calls on file content without validation expose the application to runtime exceptions from malformed JSON, corrupted files, or unexpected data structures, potentially causing settings management failures and data loss in VSCode configuration workflows.

## Decision

1. MAY: Validation logic MAY use schema validation libraries (e.g., Zod, Joi) for comprehensive type checking

## Policy Block

- MAY Validation logic MAY use schema validation libraries (e.g., Zod, Joi) for comprehensive type checking

In scope:
- All readVSCodeSettings operations that parse JSON from filesystem
- All writeVSCodeSettings operations that serialize configuration data
- Cache layer operations (existingServerMap.set) that store parsed configuration
- Public API contracts (VSCodeSettings, AugmentMcpServer) exposed to consumers

Out of scope:
- JSON parsing in test fixtures or mock data
- JSON operations on trusted internal data structures already validated
- Third-party library internal JSON operations

Exceptions:
- EXC-001: JSON content originates from trusted internal sources with guaranteed schema compliance

## Rationale

- The IR evidence shows direct JSON.parse(content) usage in settings management without visible validation, creating a vulnerability to malformed input
- With 91% confidence from src/vscode/settings.ts, the pattern is well-established in the primary datastore layer for VSCode configuration
- The cache layer pattern (existingServerMap.set) depends on validated data to maintain consistency; invalid parsed data corrupts the cache
- Public API contracts expose settings operations to external consumers who may trigger reads on corrupted or user-modified configuration files

## Consequences

Positive:
- Prevents runtime crashes from malformed JSON in VSCode settings files
- Enables graceful degradation with fallback to default configuration when settings are corrupted
- Improves debugging through explicit validation error messages with context
- Protects cache layer integrity by ensuring only validated data enters existingServerMap

Negative:
- Adds validation overhead to every settings read operation, increasing latency
- Requires additional code for validation logic and error handling paths
- May require schema definition maintenance as VSCodeSettings and AugmentMcpServer types evolve
- Fallback behavior may mask underlying file corruption issues if not properly logged

## Alternatives

- Continue using unvalidated JSON.parse() and rely on try-catch at call sites (rejected)
  Rejected because: Inconsistent error handling across call sites and no schema validation; cache corruption risk remains unmitigated
  When valid: Only acceptable for internal trusted data sources with guaranteed schema compliance
- Use schema validation library (Zod/Joi) for comprehensive type checking (deferred)
  Rejected because: Adds external dependency; can be adopted incrementally after basic validation is in place
  When valid: When settings schema complexity increases or runtime type safety becomes critical
- Implement custom validation functions matching TypeScript type definitions (accepted)
  When valid: Provides balance between safety and minimal dependencies; aligns with existing TypeScript contracts

## Risks

- Validation logic may diverge from TypeScript type definitions, creating false sense of security
  Mitigation: Generate validation functions from TypeScript types using tools like ts-to-zod or maintain co-located type guards
  Owner: Engineering team
- Overly strict validation may reject valid but unexpected configuration variations
  Mitigation: Use permissive validation for optional fields and version validation logic alongside schema evolution
  Owner: Engineering team
- Performance degradation on large settings files with complex validation
  Mitigation: Profile validation performance and implement caching of validated results based on file modification time
  Owner: Engineering team

## Implementation Notes

- Add type guard functions for VSCodeSettings and AugmentMcpServer that validate required fields and types
- Wrap JSON.parse(content) in try-catch blocks within readVSCodeSettings, returning default configuration on parse or validation failure
- Log validation failures with file path, error type, and partial content (sanitized) to aid debugging
- Update writeVSCodeSettings to validate data before serialization to prevent writing invalid configuration
- Consider adding file integrity checks (checksums) for critical settings files to detect corruption early

## Continuation Context


Verify commands:
- grep -n 'JSON\.parse' src/vscode/settings.ts | grep -v 'try\|catch' # Should return no unprotected JSON.parse calls
- grep -n 'function.*validate.*Settings\|isValid.*Settings' src/vscode/settings.ts # Should find validation functions
- npm test -- --grep 'settings.*validation.*malformed' # Should pass tests for malformed JSON handling

Accept when:
- All JSON.parse operations in src/vscode/settings.ts are wrapped in try-catch blocks
- Type guard functions exist for VSCodeSettings and AugmentMcpServer with required field validation
- Tests demonstrate graceful handling of malformed JSON with fallback to defaults and error logging

## Enforcement

- Verified by: Code review checklist requiring validation for all new JSON parsing operations
- Verified by: Static analysis rules detecting unprotected JSON.parse calls in datastore layer
- Verified by: Unit tests covering malformed input scenarios for all settings read operations
- Violation handling: CI pipeline fails if unprotected JSON.parse detected in settings management code
- Violation handling: Code review blocks merge if validation logic is missing for new parsing operations
- Violation handling: Runtime monitoring alerts on repeated JSON parse exceptions in production
- Exception process: Document exception request with source trust boundary analysis and schema guarantee mechanism
- Exception process: Tech lead reviews and approves exception with explicit risk acceptance
- Exception process: Add inline comment with exception ID and justification at exception site