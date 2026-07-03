# Validate External JSON Input with JSON.parse in Test and Agent Configuration Handlers: Agent Implementations Validate

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all agent configuration handlers and test harnesses that process external JSON configuration files.

## Context

- The codebase processes external JSON configuration files from multiple sources including .ruler/mcp.json, .vscode/mcp.json, .cursor/mcp.json, .gemini/settings.json, and .idx/mcp.json for agent configuration management
- Agent configuration handlers (OpenCodeAgent, MistralVibeAgent, AmazonQCliAgent) and test suites must parse and validate JSON content read from the filesystem before processing server definitions, context file names, and MCP server configurations
- The UnifiedConfigLoader and multiple test harnesses use JSON.parse to validate configuration structure after reading files with fs.readFile, establishing a consistent pattern across 18 files
- Configuration files contain critical runtime parameters including command paths, arguments, URLs, and server definitions that must be structurally valid before being merged or applied to agent-specific formats
- The test framework validates idempotent configuration application, backup prevention, and agent-specific key usage (servers vs mcpServers) requiring reliable JSON parsing to assert expected structure

## Problem Statement

Agent configuration handlers and test harnesses must reliably validate the structural integrity of external JSON configuration files before processing their contents. Without explicit parsing validation, malformed JSON could propagate through the configuration pipeline, causing runtime errors during agent initialization, configuration merging, or test assertion failures. The system requires a consistent approach to input validation that fails fast on invalid JSON while providing clear error boundaries between file I/O and configuration processing.

## Decision

1. SHOULD: Agent implementations SHOULD validate JSON structure immediately after parsing and before accessing nested properties like mcpServers, servers, or contextFileName

## Policy Block

- SHOULD Agent implementations SHOULD validate JSON structure immediately after parsing and before accessing nested properties like mcpServers, servers, or contextFileName

In scope:
- Agent configuration handlers: OpenCodeAgent, MistralVibeAgent, AmazonQCliAgent
- Core configuration loaders: UnifiedConfigLoader
- Test harnesses validating agent configuration: gemini-no-backup.test.ts, apply-mcp.firebase.test.ts, agent-specific-disable.test.ts, mcp-key-per-agent.test.ts, mcp-backup-prevention.test.ts, apply-mcp.toml-json-merge.test.ts, agent-mcp-servers.test.ts, mcp-empty-server-key-fix.test.ts, apply-mcp.overwrite.test.ts, mcp-key-gemini.test.ts
- Configuration files: .ruler/mcp.json, .vscode/mcp.json, .cursor/mcp.json, .gemini/settings.json, .idx/mcp.json, .mcp.json
- Any code path that reads and processes external JSON configuration files for agent or MCP server definitions

Out of scope:
- TOML configuration parsing (handled by @iarna/toml parseTOML)
- Internal JSON serialization for writing configuration files (JSON.stringify)
- Runtime object manipulation after initial parsing and validation
- Non-configuration JSON processing (e.g., API responses, data payloads)
- Markdown file processing (AGENTS.md, CLAUDE.md)

Exceptions:
- EXC-001: Configuration content is generated programmatically within the same process and guaranteed to be valid JSON

## Rationale

- The evidence shows 18 files consistently using JSON.parse after fs.readFile operations, establishing a proven pattern for input validation across agent configuration handlers and test harnesses
- JSON.parse provides immediate structural validation with clear error boundaries, allowing the system to fail fast on malformed configuration before attempting to access nested properties or merge server definitions
- The pattern separates filesystem I/O concerns from configuration validation concerns, enabling independent error handling for file access failures versus JSON syntax errors
- Test evidence demonstrates that JSON.parse validation is critical for asserting configuration structure, including agent-specific key usage (servers vs mcpServers), server definition presence, and idempotent configuration application

## Consequences

Positive:
- Clear error boundaries between file I/O failures and JSON syntax errors, improving debuggability when configuration files are malformed or corrupted
- Consistent validation approach across 18 files reduces cognitive load and maintenance burden when adding new agent configuration handlers
- Fast failure on invalid JSON prevents propagation of malformed configuration through the merge pipeline, protecting downstream agent initialization
- Test harnesses can reliably assert on configuration structure without defensive null checks or try-catch blocks around property access

Negative:
- JSON.parse throws synchronous exceptions that must be handled by calling code, requiring explicit error handling or allowing exceptions to propagate
- No built-in schema validation beyond JSON syntax, requiring additional validation logic for configuration structure and required properties
- Parse errors provide limited context about which configuration file or agent caused the failure without additional error wrapping
- Performance overhead of parsing large configuration files on every agent initialization, though mitigated by typically small configuration file sizes

## Alternatives

- Use a JSON schema validation library (e.g., ajv, zod) for combined parsing and schema validation (rejected)
  Rejected because: Evidence shows the codebase uses native JSON.parse without schema validation libraries, and adding schema validation would introduce new dependencies not present in the detected pattern. The current approach separates parsing from structural validation, allowing flexibility in validation logic per agent type.
  When valid: When configuration complexity increases to require formal schema definitions with nested object validation, type coercion, or default value injection
- Use JSON5 or relaxed JSON parsing to allow comments and trailing commas in configuration files (rejected)
  Rejected because: No evidence of JSON5 or relaxed parsing libraries in the detected pattern. All configuration files use strict JSON format, and introducing relaxed parsing would change the validation contract without clear evidence of user demand for comments or trailing commas.
  When valid: When user feedback indicates strong demand for human-friendly configuration file features like comments or trailing commas
- Defer JSON parsing until property access using lazy evaluation or proxy objects (rejected)
  Rejected because: Evidence shows immediate parsing after file read operations, establishing fail-fast validation. Lazy parsing would delay error detection until property access, complicating error handling and making it harder to distinguish between missing properties and malformed JSON.
  When valid: When configuration files are very large and only a small subset of properties are accessed in typical execution paths

## Risks

- Unhandled JSON.parse exceptions could crash agent initialization or test execution if calling code does not implement appropriate error handling
  Mitigation: Establish error handling conventions in agent base classes and test harnesses. Document expected exception types and provide helper functions for safe configuration loading with fallback behavior.
  Owner: Engineering team
- JSON.parse validates syntax but not semantic correctness, allowing structurally valid but semantically invalid configurations (e.g., wrong key names, missing required properties) to pass validation
  Mitigation: Implement post-parse validation checks for required properties and expected structure. Use TypeScript interfaces to document expected configuration shape and enable compile-time checking where possible.
  Owner: Engineering team
- Configuration files from untrusted sources could contain valid JSON with malicious content (e.g., command injection in server command fields)
  Mitigation: Treat all configuration file content as untrusted input. Implement additional validation for command paths, arguments, and URLs. Consider sandboxing or validation of executable paths before invoking MCP servers.
  Owner: Security team

## Implementation Notes

- Use the pattern: const config = JSON.parse(await fs.readFile(configPath, 'utf8')) for async file reads, or const config = JSON.parse(fs.readFileSync(configPath, 'utf8')) for synchronous reads
- Wrap JSON.parse calls in try-catch blocks at appropriate boundaries (e.g., agent initialization, test setup) to provide context-specific error messages that include the configuration file path
- After parsing, immediately validate the presence of expected top-level keys (e.g., mcpServers, servers, contextFileName) before accessing nested properties to provide clear error messages for structural issues
- In test harnesses, use JSON.parse validation as a precondition before assertions to ensure test failures indicate actual configuration issues rather than parse errors

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse.*fs\.readFile\|JSON\.parse.*readFile' src/agents/ tests/ --include='*.ts' | wc -l
- grep -r 'JSON\.parse' src/agents/OpenCodeAgent.ts src/agents/AmazonQCliAgent.ts src/core/UnifiedConfigLoader.ts
- npm test -- --testNamePattern='(gemini-no-backup|apply-mcp|agent-specific-disable|mcp-key)' 2>&1 | grep -c 'PASS'

Accept when:
- All agent configuration handlers in src/agents/ use JSON.parse immediately after reading configuration files with fs.readFile or fs/promises readFile
- Test harnesses validate configuration structure using JSON.parse before asserting on nested properties, server definitions, or key presence
- grep verification shows consistent JSON.parse usage across at least 15 files in agent handlers and test suites

## Enforcement

- Verified by: Code review checklist requiring JSON.parse validation for all new agent configuration handlers
- Verified by: Automated grep-based verification in CI pipeline checking for JSON.parse usage after fs.readFile operations
- Verified by: Test suite execution validating that configuration parsing tests pass with expected structure assertions
- Violation handling: Code review rejection for new agent handlers that read JSON configuration files without JSON.parse validation
- Violation handling: CI pipeline warnings when new configuration file reads are detected without corresponding JSON.parse calls
- Violation handling: Runtime errors from malformed configuration files are logged with file path and parse error details for debugging
- Exception process: Request exception through tech lead review with justification for alternative validation approach
- Exception process: Document the alternative validation mechanism and its guarantees in code comments and ADR exception log
- Exception process: Require demonstration that the alternative approach provides equivalent or superior validation guarantees