# Validate JSON Input from File System Before Parsing in Security-Critical Contexts: Json Content Read

These rules are ALWAYS ACTIVE for all code that reads configuration files from the file system and parses them as JSON or TOML, particularly in agent configuration management and MCP server handling.

### Rules

- **R-JSON-001** MUST: All JSON content read from the file system MUST be parsed using JSON.parse() with error handling that catches SyntaxError exceptions.
- **R-JSON-002** MUST: All TOML content read from the file system MUST be parsed with error handling that catches parsing exceptions.
- **R-JSON-003** MUST: After parsing configuration files, immediately validate that expected top-level keys exist (e.g., mcpServers for Cursor, servers for Copilot, contextFileName for Gemini) before use in agent configuration or MCP server definitions.
- **R-JSON-004** SHOULD: Create shared utility functions like safeParseJSON(content: string, filePath: string) and safeParseConfig(filePath: string) to centralize error handling patterns across agent implementations.
- **R-JSON-005** SHOULD: Use fs.readFile with 'utf8' encoding consistently across all configuration file reads to ensure proper string handling.
- **R-JSON-006** SHOULD: Sanitize error messages to remove absolute paths; log detailed errors only in debug mode; provide user-friendly error messages in production.

### Verify

```bash
# Detect unprotected JSON.parse calls on file system content
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l  # Should be 0 unprotected calls

# Detect unprotected TOML parsing calls
grep -r 'parseTOML' src/ tests/ | grep -v 'try\|catch' | wc -l  # Should be 0 unprotected calls

# Verify error handling tests exist for malformed configuration files
npm test -- --testNamePattern='malformed|invalid|parse.*error'
```

**Accept when:**
- All JSON.parse() and parseTOML() calls in src/ and tests/ are wrapped in try-catch blocks or called from functions that handle errors
- Parsed configuration objects are validated for expected structure before use in agent configuration or MCP server definitions
- Test suite includes cases for malformed JSON/TOML files that verify proper error handling and descriptive error messages
- Error messages do not expose sensitive file paths or configuration details in production logs

<enforcement>
Clause Code MUST NOT skip or defer verification. All file system parsing operations require error handling before merge. Security team approval required for any exceptions.
</enforcement>