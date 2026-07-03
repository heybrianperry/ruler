# Use JSON.parse for Configuration File Deserialization in Test and Agent Code: Code Use Json

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all test suites and agent implementation code that processes JSON configuration files.

## Context

- The codebase processes multiple JSON configuration files across different AI coding assistant agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, etc.) that require deserialization of settings, MCP server configurations, and agent instructions
- Test suites validate configuration file generation and merging behavior by reading and parsing JSON files from filesystem locations including .vscode/mcp.json, .gemini/settings.json, .cursor/mcp.json, .kilocode/mcp.json, .mcp.json, opencode.json, and .zed/settings.json
- Agent implementation classes (OpenCodeAgent, ZedAgent, GeminiCliAgent) read existing configuration files to merge with ruler-generated configurations, requiring reliable deserialization of user-provided JSON data
- The pattern appears consistently across 8 files with 92.67% confidence, indicating an established practice for handling JSON configuration input validation and deserialization
- Configuration files contain sensitive data including MCP server URLs, authentication tokens, command arguments, and environment variables that must be parsed correctly to prevent configuration errors or security issues

## Problem Statement

JSON configuration files from multiple sources (user-created, agent-generated, test fixtures) must be reliably deserialized into JavaScript objects for validation, merging, and processing. Inconsistent or unsafe deserialization approaches could lead to runtime errors, configuration corruption, or security vulnerabilities when handling malformed or malicious JSON input.

## Decision

1. MAY: Code MAY use JSON parsing libraries (e.g., json5, hjson) for enhanced JSON formats only when explicitly required by the configuration file format specification

## Policy Block

- MAY Code MAY use JSON parsing libraries (e.g., json5, hjson) for enhanced JSON formats only when explicitly required by the configuration file format specification

In scope:
- All test files that read and validate JSON configuration files (*.test.ts files in tests/ directory)
- All agent implementation classes that read existing JSON configuration files (src/agents/*Agent.ts)
- Configuration merging logic that combines ruler-generated and user-provided JSON configurations
- JSON files including: .vscode/mcp.json, .gemini/settings.json, .cursor/mcp.json, .kilocode/mcp.json, .mcp.json, opencode.json, .zed/settings.json, .junie/mcp/mcp.json, .qwen/settings.json

Out of scope:
- TOML configuration file parsing (uses dedicated TOML parsers)
- YAML configuration file parsing (uses dedicated YAML parsers)
- Markdown file processing (plain text, no deserialization required)
- Binary file formats
- JSON serialization (writing JSON to files using JSON.stringify)

Exceptions:
- EXC-001: A specific agent or configuration format explicitly requires a non-standard JSON parser (e.g., JSON5 for comments support)

## Rationale

- JSON.parse() is the standard, built-in JavaScript method for safely deserializing JSON strings, providing consistent behavior across Node.js versions and avoiding security risks associated with dynamic code execution
- The evidence shows 8 files consistently using JSON.parse() for configuration deserialization across test suites (mcp-empty-server-key-fix.test.ts, mcp-backup-prevention.test.ts, ruler.integration.test.ts, apply-mcp.merge.test.ts, mcp-key-gemini.test.ts, GeminiCliAgent.test.ts) and production agent code (OpenCodeAgent.ts, ZedAgent.ts)
- Using JSON.parse() uniformly across test and production code ensures test fixtures accurately represent real-world parsing behavior, improving test reliability and reducing production defects
- The pattern supports secure input validation by providing a single, auditable deserialization point where malformed JSON throws predictable exceptions rather than causing undefined behavior

## Consequences

Positive:
- Consistent, predictable JSON deserialization behavior across all configuration file processing code
- Reduced security risk by avoiding dynamic code execution methods like eval() that could execute malicious code embedded in configuration files
- Improved error handling and debugging through standardized JSON.parse() exception behavior (SyntaxError for malformed JSON)
- Better test coverage and reliability by ensuring test fixtures use the same deserialization path as production code

Negative:
- JSON.parse() only supports strict JSON format, requiring additional libraries if configuration files need comments, trailing commas, or other JSON5/relaxed JSON features
- Error messages from JSON.parse() may be cryptic for end users, requiring additional error handling and user-friendly error reporting
- No built-in schema validation in JSON.parse(), requiring separate validation logic to ensure configuration objects have expected structure and types

## Alternatives

- Use eval() or Function() constructor to deserialize JSON configuration files (rejected)
  Rejected because: Introduces critical security vulnerabilities by allowing arbitrary code execution if malicious content is embedded in configuration files. JSON.parse() provides safe deserialization without code execution risk.
  When valid: Never valid for configuration file deserialization
- Use JSON5 or hjson libraries for all JSON configuration parsing to support comments and relaxed syntax (rejected)
  Rejected because: Adds unnecessary dependencies and complexity when standard JSON format is sufficient for current configuration files. Evidence shows JSON.parse() meets current needs across 8 files.
  When valid: Valid only when specific agent configuration formats explicitly require JSON5 features (comments, trailing commas) as documented in agent specifications
- Use agent-specific parsing logic with different deserialization methods per agent type (rejected)
  Rejected because: Creates inconsistent behavior, increases maintenance burden, and makes testing more complex. Uniform JSON.parse() usage provides consistency and reliability.
  When valid: Valid only for non-JSON configuration formats (TOML, YAML) that require format-specific parsers

## Risks

- Malformed JSON in user-provided configuration files causes uncaught exceptions and application crashes
  Mitigation: Wrap all JSON.parse() calls in try-catch blocks with user-friendly error messages indicating which configuration file is malformed and providing guidance for fixing JSON syntax errors
  Owner: Engineering team
- Parsed JSON objects contain unexpected structure or missing required fields, causing runtime errors in downstream code
  Mitigation: Implement schema validation immediately after JSON.parse() using TypeScript type guards or validation libraries (e.g., zod, ajv) to verify configuration structure before use
  Owner: Engineering team
- Large or deeply nested JSON configuration files cause performance issues or stack overflow during parsing
  Mitigation: Document maximum configuration file size limits, implement file size checks before parsing, and consider streaming JSON parsers for exceptionally large configuration files if needed
  Owner: Engineering team

## Implementation Notes

- Always read configuration files using fs.readFile() or fs/promises readFile() with 'utf8' encoding before passing content to JSON.parse()
- Wrap JSON.parse() calls in try-catch blocks and provide context-specific error messages that include the configuration file path and guidance for users
- After parsing, validate that required configuration keys exist (e.g., mcpServers, contextFileName, $schema) before accessing nested properties
- In test code, use JSON.parse() on test fixture content to ensure tests validate the same deserialization behavior as production code
- Consider creating a shared utility function (e.g., parseJsonConfig(filePath)) that encapsulates file reading, JSON.parse(), error handling, and basic validation for reuse across agents

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' tests/ src/agents/ --include='*.ts' | wc -l
- grep -r 'eval\|Function(' tests/ src/agents/ --include='*.ts' | grep -v 'node_modules' || echo 'No unsafe deserialization found'
- npm test -- --testNamePattern='JSON|config|settings' --passWithNoTests

Accept when:
- All JSON configuration file deserialization in test and agent code uses JSON.parse() exclusively
- No instances of eval(), Function(), or other dynamic code execution methods are used for JSON deserialization
- Test suites successfully parse and validate JSON configuration files for all supported agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, etc.)
- Error handling is present around JSON.parse() calls to gracefully handle malformed JSON input

## Enforcement

- Verified by: Code review checklist requiring JSON.parse() for all new configuration file deserialization code
- Verified by: Static analysis or linting rules flagging use of eval() or Function() in configuration processing code
- Verified by: Integration test suite validating JSON configuration parsing across all agent implementations
- Verified by: Security audit reviewing configuration file handling for unsafe deserialization patterns
- Violation handling: Code review rejection for pull requests introducing eval() or Function() for JSON deserialization
- Violation handling: CI pipeline failure if static analysis detects unsafe deserialization patterns
- Violation handling: Mandatory refactoring of existing code found to use unsafe deserialization methods during security audits
- Exception process: Document specific technical requirement for non-standard JSON parsing (e.g., JSON5 for comment support) in architecture decision
- Exception process: Obtain architecture review approval with security assessment of alternative parsing library
- Exception process: Add inline code comments explaining exception rationale and security considerations
- Exception process: Include exception in agent-specific documentation and update policy_exceptions in this ADR