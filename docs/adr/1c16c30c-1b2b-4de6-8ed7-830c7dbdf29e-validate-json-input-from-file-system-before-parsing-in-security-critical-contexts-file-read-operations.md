# Validate JSON Input from File System Before Parsing in Security-Critical Contexts: File Read Operations

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all code that reads configuration files from the file system and parses them as JSON or TOML, particularly in agent configuration management and MCP server handling.

## Context

- The codebase manages configuration files for multiple AI agent platforms (Gemini, Copilot, Cursor, Claude, OpenCode, Amazon Q, Mistral Vibe) that require reading and parsing JSON and TOML files from the file system
- Configuration files contain sensitive MCP (Model Context Protocol) server definitions with commands, arguments, URLs, and authentication details that must be validated before use
- Test suites demonstrate extensive use of JSON.parse() on file system content across 18 files, with pattern detection showing 92.72% confidence in this practice
- The system performs merge operations between ruler-managed configurations and native agent configurations, requiring reliable parsing of potentially malformed or corrupted files
- File system operations include reading settings from .gemini/settings.json, .vscode/mcp.json, .cursor/mcp.json, .idx/mcp.json, and .ruler/mcp.json paths

## Problem Statement

Configuration files read from the file system may contain malformed JSON or TOML content due to manual editing, corruption, incomplete writes, or malicious modification. Parsing these files without validation can lead to runtime exceptions, security vulnerabilities through injection attacks, or silent failures that compromise agent configuration integrity. The system needs a consistent approach to validate and safely parse configuration data before use.

## Decision

1. MUST: File read operations using fs.readFile() or fs/promises MUST specify 'utf8' encoding when reading configuration files intended for parsing

## Policy Block

- MUST File read operations using fs.readFile() or fs/promises MUST specify 'utf8' encoding when reading configuration files intended for parsing

In scope:
- All agent configuration files in .gemini/, .vscode/, .cursor/, .idx/, .ruler/ directories
- MCP server configuration files (mcp.json, settings.json)
- TOML configuration files (ruler.toml)
- Test harness code that reads and validates configuration files
- Agent implementation classes (OpenCodeAgent, MistralVibeAgent, AmazonQCliAgent, etc.)
- UnifiedConfigLoader and configuration merge logic

Out of scope:
- JSON parsing of HTTP response bodies from external APIs
- JSON serialization for writing configuration files (JSON.stringify)
- In-memory JSON manipulation that does not originate from file system reads
- Binary file formats or non-configuration data files

Exceptions:
- EXC-001: Test fixtures with intentionally malformed JSON to verify error handling behavior

## Rationale

- The evidence shows consistent use of JSON.parse() across 18 files with 92.72% confidence, indicating an established pattern that requires formalization for security and reliability
- Configuration files control agent behavior and MCP server execution, making them security-critical assets that require validated parsing to prevent command injection or privilege escalation
- The codebase uses multiple parsing libraries (JSON.parse for JSON, parseTOML from @iarna/toml for TOML) that require consistent error handling patterns
- Test evidence demonstrates validation of parsed objects for expected keys (mcpServers, servers, contextFileName) indicating awareness of structure validation needs

## Consequences

Positive:
- Prevents runtime crashes from malformed configuration files by enforcing error handling at parse boundaries
- Reduces security risk by validating configuration structure before executing commands or URLs from MCP server definitions
- Improves debugging experience by catching parse errors early with descriptive error messages
- Establishes consistent parsing patterns across all agent implementations and configuration loaders

Negative:
- Adds error handling boilerplate to every file read and parse operation
- May increase initial implementation time for new agent integrations
- Requires maintenance of validation logic as configuration schemas evolve
- Could mask underlying file system issues if error handling is too permissive

## Alternatives

- Use a centralized configuration parsing service that handles all file reads and validation (rejected)
  Rejected because: Would require significant refactoring of existing agent implementations and test harness code, and adds unnecessary abstraction for simple parse operations
  When valid: In a greenfield project or during a major architecture refactor where centralized configuration management provides additional benefits
- Rely on TypeScript type checking without runtime validation (rejected)
  Rejected because: TypeScript types are erased at runtime and provide no protection against malformed JSON from file system, which is the primary threat vector
  When valid: Never valid for file system input validation, though types remain useful for compile-time safety
- Implement schema validation using Zod or similar library for all configuration files (deferred)
  Rejected because: Adds dependency and complexity; current pattern of structural validation (checking for expected keys) is sufficient for immediate needs
  When valid: When configuration schemas become more complex or when formal schema documentation is required for external integrations

## Risks

- Inconsistent error handling across different agent implementations may leave some code paths vulnerable to parse failures
  Mitigation: Establish shared utility functions for configuration parsing with standardized error handling; add linting rules to detect unprotected JSON.parse calls
  Owner: Engineering team
- Overly strict validation may reject valid but unexpected configuration formats, breaking backward compatibility
  Mitigation: Use permissive structural validation (check for required keys but allow additional properties); version configuration schemas explicitly
  Owner: Engineering team
- Error messages from parse failures may expose sensitive file paths or configuration details in logs
  Mitigation: Sanitize error messages to remove absolute paths; log detailed errors only in debug mode; provide user-friendly error messages in production
  Owner: Security team

## Implementation Notes

- Wrap all JSON.parse() calls in try-catch blocks that catch SyntaxError and provide context about which configuration file failed to parse
- After parsing, immediately validate that expected top-level keys exist (e.g., mcpServers for Cursor, servers for Copilot, contextFileName for Gemini)
- Use fs.readFile with 'utf8' encoding consistently across all configuration file reads to ensure proper string handling
- Consider creating shared utility functions like safeParseJSON(content: string, filePath: string) and safeParseConfig(filePath: string) to centralize error handling patterns
- In test code, use expect().toThrow() or similar assertions to verify that malformed configuration files are properly rejected

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l  # Should be 0 unprotected calls
- grep -r 'parseTOML' src/ tests/ | grep -v 'try\|catch' | wc -l  # Should be 0 unprotected calls
- npm test -- --testNamePattern='malformed|invalid|parse.*error'  # Verify error handling tests exist

Accept when:
- All JSON.parse() and parseTOML() calls in src/ and tests/ are wrapped in try-catch blocks or called from functions that handle errors
- Parsed configuration objects are validated for expected structure before use in agent configuration or MCP server definitions
- Test suite includes cases for malformed JSON/TOML files that verify proper error handling and descriptive error messages

## Enforcement

- Verified by: Code review checklist requiring error handling for all file system parsing operations
- Verified by: ESLint custom rule to detect unprotected JSON.parse() calls on file system content
- Verified by: Test coverage requirements for configuration parsing error paths
- Violation handling: CI pipeline fails if grep commands detect unprotected parse calls
- Violation handling: Code review blocks merge if configuration parsing lacks error handling
- Violation handling: Security review required for any new agent implementation before merge
- Exception process: Document the specific reason why error handling is not required in code comments
- Exception process: Obtain approval from security team for any exception to parsing validation rules
- Exception process: Add exception to .eslintrc with justification comment