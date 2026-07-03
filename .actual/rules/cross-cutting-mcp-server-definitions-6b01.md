# Validate JSON Input from File System Before Parsing in Security-Critical Contexts: Mcp Server Definitions

These rules are ALWAYS ACTIVE for all code that reads configuration files from the file system and parses them as JSON or TOML, particularly in agent configuration management and MCP server handling across .gemini/, .vscode/, .cursor/, .idx/, .ruler/ directories and test harness code.

### Rules

- **R-MCP-001** MUST: Wrap all JSON.parse() calls in try-catch blocks that catch SyntaxError and provide context about which configuration file failed to parse.
- **R-MCP-002** MUST: Wrap all parseTOML() calls in try-catch blocks that catch parsing errors and provide context about which configuration file failed to parse.
- **R-MCP-003** SHOULD: MCP server definitions SHOULD be validated for required fields (command, args, url, type) after parsing and before merging or applying.
- **R-MCP-004** SHOULD: After parsing configuration files, immediately validate that expected top-level keys exist (e.g., mcpServers for Cursor, servers for Copilot, contextFileName for Gemini).
- **R-MCP-005** SHOULD: Use fs.readFile with 'utf8' encoding consistently across all configuration file reads to ensure proper string handling.
- **R-MCP-006** MAY: Consider creating shared utility functions like safeParseJSON(content: string, filePath: string) and safeParseConfig(filePath: string) to centralize error handling patterns.
- **R-MCP-007** SHOULD: Sanitize error messages to remove absolute paths; log detailed errors only in debug mode; provide user-friendly error messages in production.

### Verify

```bash
# Detect unprotected JSON.parse calls in source and test files
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l  # Should be 0 unprotected calls

# Detect unprotected parseTOML calls in source and test files
grep -r 'parseTOML' src/ tests/ | grep -v 'try\|catch' | wc -l  # Should be 0 unprotected calls

# Verify error handling tests exist for malformed configuration files
npm test -- --testNamePattern='malformed|invalid|parse.*error'
```

**Accept when:**
- All JSON.parse() and parseTOML() calls in src/ and tests/ are wrapped in try-catch blocks or called from functions that handle errors
- Parsed configuration objects are validated for expected structure (required fields and top-level keys) before use in agent configuration or MCP server definitions
- Test suite includes cases for malformed JSON/TOML files that verify proper error handling and descriptive error messages
- Error messages are sanitized to avoid exposing sensitive file paths or configuration details in logs

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All JSON.parse() and parseTOML() calls on file system content must be protected by error handling before merge. Security review is required for any new agent implementation.
</enforcement>