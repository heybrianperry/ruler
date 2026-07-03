# Adopt JSON.parse for External Configuration Input Validation: Implementations Use Json

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all configuration loading, agent integration, and external data parsing operations within the Ruler CLI codebase.

## Context

- The Ruler CLI system integrates with multiple AI coding assistants (Claude, Copilot, Cursor, Gemini, OpenCode, Zed, etc.) that use JSON-based configuration files for MCP servers, settings, and agent instructions
- Configuration files are read from filesystem locations (.mcp.json, settings.json, opencode.json, etc.) and must be parsed to extract existing server definitions, merge strategies, and agent-specific settings
- Test suites validate configuration generation and merging behavior by parsing generated JSON outputs to verify correctness of MCP server configurations, schema compliance, and content integrity
- The codebase also processes TOML configuration files (ruler.toml, config.toml) using @iarna/toml parser, establishing a pattern of using standard library parsers for structured data validation
- Input validation occurs at multiple boundaries: file I/O operations, agent configuration loading, test assertions, and MCP server configuration merging

## Problem Statement

Without standardized input validation for external configuration data, the system risks runtime failures from malformed JSON, inconsistent error handling across agent integrations, and security vulnerabilities from unvalidated external input. The codebase must reliably parse configuration files from diverse sources while providing clear failure modes and maintaining type safety.

## Decision

1. MAY: Implementations MAY use JSON.stringify() with formatting options when writing configuration files to maintain readability

## Policy Block

- MAY Implementations MAY use JSON.stringify() with formatting options when writing configuration files to maintain readability

In scope:
- All agent configuration files (.mcp.json, settings.json, opencode.json, .gemini/settings.json, .qwen/settings.json, .zed/settings.json, etc.)
- MCP server configuration parsing in agent implementations (FirebenderAgent, GeminiCliAgent, etc.)
- Test harness validation of generated configuration outputs
- VSCode settings.json parsing for AugmentCode MCP server integration
- Backup file restoration operations that read .bak JSON files

Out of scope:
- TOML configuration parsing (uses @iarna/toml library per separate pattern)
- YAML configuration parsing (uses dedicated YAML parser)
- Markdown content processing (plain text, not structured data)
- Command-line argument parsing (uses CLI framework)
- Environment variable processing (string-based, not JSON)

Exceptions:
- EXC-001: Legacy configuration files contain non-standard JSON extensions (comments, trailing commas) that require preprocessing

## Rationale

- JSON.parse() is the standard Node.js built-in method for parsing JSON, providing consistent error handling and security guarantees without external dependencies
- The pattern appears in 17 files with 92.14% confidence, indicating widespread adoption across agent integrations, test suites, and core configuration loading logic
- Using JSON.parse() enables immediate syntax validation at parse time, preventing downstream errors from malformed configuration that would be harder to diagnose
- The approach aligns with the existing pattern of using standard library parsers (@iarna/toml for TOML files) rather than custom parsing logic

## Consequences

Positive:
- Consistent error handling across all agent configuration loading paths with clear parse failure messages
- Built-in protection against malformed JSON preventing runtime type errors in configuration processing
- Zero external dependencies for JSON parsing reduces supply chain risk and bundle size
- Test suites can reliably validate generated configuration files match expected schemas

Negative:
- JSON.parse() throws synchronous exceptions requiring try-catch blocks throughout the codebase
- No built-in schema validation requires additional validation layer (zod) for type safety
- Error messages from JSON.parse() may not be user-friendly for non-technical users editing configuration files
- Cannot handle JSON extensions (comments, trailing commas) that some editors may introduce

## Alternatives

- Use a third-party JSON parser library (e.g., json5, hjson) that supports extended JSON syntax with comments and trailing commas (rejected)
  Rejected because: Introduces external dependency for marginal benefit; standard JSON is sufficient for configuration files and widely supported by all agent tools
  When valid: If multiple agent tools require non-standard JSON features and cannot be migrated to standard JSON
- Use fs.readFile with 'json' encoding or a streaming JSON parser for large configuration files (rejected)
  Rejected because: Configuration files are small (typically <100KB); streaming adds complexity without performance benefit; Node.js does not support 'json' encoding natively
  When valid: If configuration files grow to multi-megabyte size requiring streaming to avoid memory pressure
- Implement custom JSON parser with enhanced error reporting showing line/column of syntax errors (deferred)
  Rejected because: Significant implementation effort; JSON.parse() errors are sufficient for developer use cases; could be added as enhancement layer
  When valid: If user feedback indicates parse error messages are a significant usability barrier for non-developers

## Risks

- Malformed JSON in user-edited configuration files causes parse failures that block agent configuration application
  Mitigation: Wrap JSON.parse() in try-catch with clear error messages indicating file path and suggesting JSON validation tools; consider pre-flight validation command
  Owner: Engineering team
- JSON.parse() accepts any valid JSON including unexpected types (arrays instead of objects) leading to runtime errors in downstream code
  Mitigation: Implement schema validation layer using zod immediately after parsing; validate expected top-level structure before accessing properties
  Owner: Engineering team
- Large or deeply nested JSON configuration files could cause performance issues or stack overflow in JSON.parse()
  Mitigation: Document maximum configuration file size limits; add file size checks before parsing; monitor parse performance in production
  Owner: Engineering team

## Implementation Notes

- Always read file content as UTF-8 string using fs.readFile(path, 'utf8') before passing to JSON.parse()
- Wrap JSON.parse() calls in try-catch blocks with error handling that logs file path and original error message
- Use zod schemas (as seen in ConfigLoader.ts) to validate parsed JSON structure matches expected agent configuration format
- In test suites, use JSON.parse() to verify generated files are valid JSON before asserting on specific fields
- For agent implementations, follow the pattern in GeminiCliAgent.ts and FirebenderAgent.ts of parsing existing config, merging with new values, and writing back with JSON.stringify()

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' src/ tests/ | wc -l  # Should show widespread usage across codebase
- grep -r 'JSON\.parse' src/agents/ | grep -v 'try\|catch' && echo 'Found unprotected JSON.parse' || echo 'All JSON.parse calls protected'
- npm test -- --testNamePattern='should.*parse.*json|JSON' | grep -E 'PASS|FAIL'  # Verify JSON parsing tests pass

Accept when:
- All agent configuration loading code uses JSON.parse() wrapped in try-catch blocks with error logging
- Test suites successfully parse and validate generated JSON configuration files for all supported agents
- No instances of eval() or Function() constructor used for parsing configuration data
- Schema validation (zod) is applied to parsed JSON objects before accessing nested properties in production code

## Enforcement

- Verified by: Automated test suite validates JSON parsing behavior across 17 test files with describe/it blocks
- Verified by: Code review checklist includes verification that new configuration loading uses JSON.parse() with error handling
- Verified by: Static analysis (ESLint) configured to flag eval() usage and recommend JSON.parse() for JSON data
- Violation handling: CI pipeline fails if tests detect malformed JSON in generated configuration files
- Violation handling: Code review blocks merge if JSON parsing lacks try-catch error handling
- Violation handling: Runtime errors from JSON.parse() failures are logged with file path and surfaced to user with actionable guidance
- Exception process: Request exception through GitHub issue documenting specific agent requirement for non-standard JSON
- Exception process: Tech lead reviews exception request and approves only if agent tool cannot support standard JSON
- Exception process: Approved exceptions must document preprocessing step and include migration plan to standard JSON