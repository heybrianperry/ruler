# Enforce JSON.parse Input Validation for External Configuration Data: Implementations Use Typescript

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all agent configuration processing, MCP server configuration handling, and external JSON file parsing operations within the Ruler CLI tool and its test suite.

## Context

- The Ruler CLI tool processes configuration files from multiple AI coding assistant agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Claude, and others), requiring consistent parsing of JSON configuration data from filesystem sources.
- MCP (Model Context Protocol) server configurations are read from various agent-specific paths (.vscode/mcp.json, .cursor/mcp.json, .gemini/settings.json, .zed/settings.json, opencode.json) and must be validated before merging or applying transformations.
- Test harness code validates agent configuration outputs by parsing JSON results from file system operations, requiring reliable deserialization to verify correct behavior across 8+ test files.
- Configuration files contain sensitive data including server URLs, authentication tokens, environment variables, and command execution parameters that must be validated before use.
- The codebase handles both stdio-based local MCP servers and remote API endpoints, requiring consistent validation across different configuration schemas and agent implementations.

## Problem Statement

Without systematic input validation of JSON configuration data, the Ruler CLI tool and its agent implementations are vulnerable to malformed configuration files, injection attacks through crafted JSON payloads, and runtime failures from unexpected data structures. The tool must safely parse external JSON from multiple agent configuration formats while maintaining type safety and preventing security vulnerabilities.

## Decision

1. MAY: Implementations MAY use TypeScript interfaces or JSON schema validation libraries to enforce configuration structure beyond basic JSON.parse() validation.

## Policy Block

- MAY Implementations MAY use TypeScript interfaces or JSON schema validation libraries to enforce configuration structure beyond basic JSON.parse() validation.

In scope:
- All agent implementation classes (OpenCodeAgent, ZedAgent, GeminiCliAgent, AgentsMdAgent, AbstractAgent) that read JSON configuration files
- MCP configuration processing in applyRulerConfig methods across all agent types
- Test harness code that validates agent output by parsing JSON from filesystem operations
- Configuration merge operations that combine .ruler/mcp.json with agent-native configuration files
- Settings file readers for .gemini/settings.json, .qwen/settings.json, .zed/settings.json, opencode.json, and other JSON-based agent configurations

Out of scope:
- JSON serialization (JSON.stringify) operations for writing configuration files
- TOML configuration parsing for .codex/config.toml and config.toml files
- YAML configuration parsing for .aider.conf.yml files
- Markdown content processing for AGENTS.md, CLAUDE.md, and other instruction files
- In-memory JSON object manipulation that does not involve external input

Exceptions:
- EXC-001: Test fixtures with known-valid JSON content may skip explicit error handling when the test setup guarantees valid JSON structure
- EXC-002: Internal JSON parsing of trusted, programmatically-generated configuration may use simplified validation if the generator is covered by unit tests

## Rationale

- The evidence shows 8 files with consistent JSON.parse() usage across test files (mcp-empty-server-key-fix.test.ts, mcp-backup-prevention.test.ts, ruler.integration.test.ts, apply-mcp.merge.test.ts, mcp-key-gemini.test.ts) and agent implementations (OpenCodeAgent.ts, ZedAgent.ts, GeminiCliAgent.test.ts), indicating an established pattern of input validation.
- MCP server configurations contain command execution parameters (command, args fields) and network endpoints (url, headers) that require validation to prevent injection attacks and unauthorized access.
- The pattern detection shows 92.67% confidence across multiple agent types (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Claude, Aider, Junie, Kilo Code) demonstrating consistent security practices across the codebase.
- Test evidence shows validation of both successful parsing (expect(config[key]).toBeDefined()) and structure verification (expect(typeof mcpServers).toBe('object')), indicating defense-in-depth validation beyond basic JSON parsing.

## Consequences

Positive:
- Prevents runtime crashes from malformed JSON configuration files by catching SyntaxError exceptions during parsing
- Reduces attack surface for injection vulnerabilities by validating configuration data before use in command execution or network requests
- Improves debuggability by failing fast with clear error messages when configuration files are corrupted or invalid
- Enables safe configuration merging across multiple agent formats by ensuring structural consistency before transformation operations

Negative:
- Adds parsing overhead and error handling complexity to every configuration read operation
- Requires maintenance of validation logic across multiple agent implementations with different schema requirements
- May produce verbose error handling code that obscures core configuration logic
- Increases test complexity by requiring validation of both success and failure paths for JSON parsing

## Alternatives

- Use JSON schema validation libraries (e.g., ajv, zod) for comprehensive structural validation beyond JSON.parse() (deferred)
  When valid: Consider for future enhancement if configuration complexity increases or if type safety requirements demand compile-time schema validation
- Trust filesystem configuration without validation, relying on agent tools to write valid JSON (rejected)
  Rejected because: Exposes the tool to malformed configurations from manual edits, corrupted files, or malicious actors with filesystem access. The evidence shows explicit validation in 8 files, indicating this approach was consciously avoided.
  When valid: Never valid for external configuration files
- Implement a centralized configuration parser service that handles all JSON validation in one location (rejected)
  Rejected because: Each agent has unique schema requirements (mcpServers vs servers vs context_servers keys) that require agent-specific validation logic. Centralization would create a complex abstraction that obscures agent-specific requirements.
  When valid: Consider if common validation patterns emerge across 80%+ of agent implementations

## Risks

- Inconsistent validation across agent implementations may leave some configuration paths vulnerable to malformed input
  Mitigation: Establish shared validation utilities in FileSystemUtils module and require code review verification that all JSON.parse() calls include error handling
  Owner: Engineering team - agent implementation maintainers
- JSON.parse() alone does not validate semantic correctness (e.g., valid URLs, allowed command values), leaving room for logical vulnerabilities
  Mitigation: Implement secondary validation for security-sensitive fields (command, args, url, headers) with allowlists and sanitization
  Owner: Security review team
- Error messages from JSON parsing failures may expose filesystem paths or configuration structure to attackers
  Mitigation: Sanitize error messages in production builds to avoid information disclosure while preserving debugging information in development mode
  Owner: Engineering team - error handling module

## Implementation Notes

- Wrap all fs.readFile() + JSON.parse() sequences in try-catch blocks that handle both filesystem errors and JSON SyntaxError exceptions
- For agent implementations, create typed interfaces for expected configuration structures and validate parsed objects against these interfaces before property access
- In test code, use expect() assertions on parsed JSON structure (e.g., expect(config.mcpServers).toBeDefined()) to verify both parsing success and schema correctness
- Consider extracting common validation patterns into FileSystemUtils helper functions (e.g., readJsonConfig<T>(path): T) that encapsulate error handling and type validation
- Document expected JSON schemas in agent implementation files or separate schema documentation to guide validation logic maintenance

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' src/agents/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'
- grep -r 'JSON\.parse.*await fs\.readFile' src/ tests/ | grep -v '\.test\.' | wc -l
- npm test -- --testNamePattern='JSON|parse|config' --passWithNoTests

Accept when:
- All JSON.parse() calls in src/agents/ are wrapped in try-catch blocks or use helper functions with error handling
- Test files demonstrate validation of parsed JSON structure using expect() assertions on required keys (mcpServers, servers, context_servers)
- Agent implementations validate configuration object structure before accessing nested properties or applying transformations
- Security-sensitive fields (command, args, url) undergo secondary validation beyond JSON parsing

## Enforcement

- Verified by: Code review checklist item: Verify JSON.parse() calls include error handling
- Verified by: Static analysis: grep-based verification that JSON.parse() appears within try-catch blocks
- Verified by: Unit tests: Verify agent implementations handle malformed JSON gracefully
- Verified by: Integration tests: Validate end-to-end configuration processing with invalid JSON fixtures
- Violation handling: Code review rejection for new JSON.parse() calls without error handling
- Violation handling: CI pipeline failure if grep-based verification detects unprotected JSON.parse() calls
- Violation handling: Security review escalation for violations in command execution or network request paths
- Violation handling: Mandatory refactoring for existing violations discovered during maintenance
- Exception process: Document exception rationale in code comments explaining why validation is unnecessary
- Exception process: Obtain security team approval for exceptions in security-sensitive code paths
- Exception process: Add exception to policy_exceptions section with clear allowed_when conditions
- Exception process: Review exceptions quarterly to determine if they can be eliminated