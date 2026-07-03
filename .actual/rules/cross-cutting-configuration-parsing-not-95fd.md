# Validate JSON Input from File System Before Parsing in Security-Critical Contexts: Configuration Parsing Not

These rules are ALWAYS ACTIVE for all code that reads configuration files from the file system and parses them as JSON or TOML, particularly in agent configuration management and MCP server handling across .gemini/, .vscode/, .cursor/, .idx/, and .ruler/ directories.

### Rules

- **R-CONFIG-001** MUST_NOT: Configuration parsing MUST NOT use eval() or Function() constructors to process file content.
- **R-CONFIG-002** MUST: All JSON.parse() calls on file system content MUST be wrapped in try-catch blocks that catch SyntaxError and provide context about which configuration file failed to parse.
- **R-CONFIG-003** MUST: All parseTOML() calls on file system content MUST be wrapped in try-catch blocks with proper error handling.
- **R-CONFIG-004** MUST: After parsing configuration files, immediately validate that expected top-level keys exist (e.g., mcpServers for Cursor, servers for Copilot, contextFileName for Gemini) before use in agent configuration or MCP server definitions.
- **R-CONFIG-005** SHOULD: Use fs.readFile with 'utf8' encoding consistently across all configuration file reads to ensure proper string handling.
- **R-CONFIG-006** SHOULD: Create and use shared utility functions like safeParseJSON(content: string, filePath: string) and safeParseConfig(filePath: string) to centralize error handling patterns across agent implementations.
- **R-CONFIG-007** SHOULD: Sanitize error messages to remove absolute paths; log detailed errors only in debug mode; provide user-friendly error messages in production.
- **R-CONFIG-008** SHOULD: In test code, use expect().toThrow() or similar assertions to verify that malformed configuration files are properly rejected.

### Verify

```bash
# Detect unprotected JSON.parse calls on file system content
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l  # Should be 0 unprotected calls

# Detect unprotected parseTOML calls
grep -r 'parseTOML' src/ tests/ | grep -v 'try\|catch' | wc -l  # Should be 0 unprotected calls

# Verify error handling tests exist for malformed configurations
npm test -- --testNamePattern='malformed|invalid|parse.*error'
```

**Accept when:**
- All JSON.parse() and parseTOML() calls in src/ and tests/ are wrapped in try-catch blocks or called from functions that handle errors
- Parsed configuration objects are validated for expected structure before use in agent configuration or MCP server definitions
- Test suite includes cases for malformed JSON/TOML files that verify proper error handling and descriptive error messages
- Error messages do not expose sensitive file paths or configuration details in production logs
- Shared utility functions for configuration parsing are used consistently across agent implementations

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing operations must be validated before merge. Security review is required for any new agent implementation before merge. Violations block CI pipeline and code review.
</enforcement>