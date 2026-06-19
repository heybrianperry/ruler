# Validate JSON Input from File System Before Parsing in Security-Critical Contexts: Implementations Use Schema

These rules are ALWAYS ACTIVE for all code that reads configuration files from the file system and parses them as JSON or TOML, particularly in agent configuration management and MCP server handling.

### Rules

- **R-CONFIG-001** MUST: Wrap all `JSON.parse()` calls in try-catch blocks that catch `SyntaxError` and provide context about which configuration file failed to parse.
- **R-CONFIG-002** MUST: Wrap all `parseTOML()` calls in try-catch blocks with proper error handling and context.
- **R-CONFIG-003** MUST: After parsing, immediately validate that expected top-level keys exist (e.g., `mcpServers` for Cursor, `servers` for Copilot, `contextFileName` for Gemini) before use in agent configuration or MCP server definitions.
- **R-CONFIG-004** SHOULD: Use `fs.readFile` with `'utf8'` encoding consistently across all configuration file reads to ensure proper string handling.
- **R-CONFIG-005** SHOULD: Create and use shared utility functions like `safeParseJSON(content: string, filePath: string)` and `safeParseConfig(filePath: string)` to centralize error handling patterns.
- **R-CONFIG-006** MAY: Implementations MAY use schema validation libraries (e.g., Zod, Joi) to enforce configuration structure after parsing.
- **R-CONFIG-007** MUST: Sanitize error messages to remove absolute paths; log detailed errors only in debug mode; provide user-friendly error messages in production.

### Verify

```bash
# Detect unprotected JSON.parse calls on file system content
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l  # Should be 0 unprotected calls

# Detect unprotected parseTOML calls
grep -r 'parseTOML' src/ tests/ | grep -v 'try\|catch' | wc -l  # Should be 0 unprotected calls

# Verify error handling tests exist for malformed configuration files
npm test -- --testNamePattern='malformed|invalid|parse.*error'
```

**Accept when:**
- All `JSON.parse()` and `parseTOML()` calls in `src/` and `tests/` are wrapped in try-catch blocks or called from functions that handle errors.
- Parsed configuration objects are validated for expected structure before use in agent configuration or MCP server definitions.
- Test suite includes cases for malformed JSON/TOML files that verify proper error handling and descriptive error messages.
- Error messages do not expose absolute file paths or sensitive configuration details in production logs.

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON and TOML parsing from the file system MUST be protected with error handling and structural validation before use.
</enforcement>