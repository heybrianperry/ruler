# Validate JSON Input from File System Before Parsing in Security-Critical Contexts: File Read Operations

These rules are ALWAYS ACTIVE for all code that reads configuration files from the file system and parses them as JSON or TOML, particularly in agent configuration management and MCP server handling across .gemini/, .vscode/, .cursor/, .idx/, .ruler/ directories and test harness code.

### Rules

- **R-FILE-READ-001** MUST: File read operations using fs.readFile() or fs/promises MUST specify 'utf8' encoding when reading configuration files intended for parsing.
- **R-FILE-READ-002** MUST: All JSON.parse() calls on file system content MUST be wrapped in try-catch blocks that catch SyntaxError and provide context about which configuration file failed to parse.
- **R-FILE-READ-003** MUST: All parseTOML() calls on file system content MUST be wrapped in try-catch blocks with error handling.
- **R-FILE-READ-004** MUST: After parsing configuration files, immediately validate that expected top-level keys exist (e.g., mcpServers for Cursor, servers for Copilot, contextFileName for Gemini) before use in agent configuration or MCP server definitions.
- **R-FILE-READ-005** SHOULD: Create and use shared utility functions like safeParseJSON(content: string, filePath: string) and safeParseConfig(filePath: string) to centralize error handling patterns across agent implementations.
- **R-FILE-READ-006** SHOULD: Sanitize error messages to remove absolute paths; log detailed errors only in debug mode; provide user-friendly error messages in production.

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
- All JSON.parse() and parseTOML() calls in src/ and tests/ are wrapped in try-catch blocks or called from functions that handle errors
- Parsed configuration objects are validated for expected structure before use in agent configuration or MCP server definitions
- Test suite includes cases for malformed JSON/TOML files that verify proper error handling and descriptive error messages
- File read operations use 'utf8' encoding consistently across all configuration file reads
- Error messages do not expose absolute file paths or sensitive configuration details in production logs

<enforcement>
Claude Code MUST NOT skip or defer verification. All file system parsing operations MUST comply with these rules before code review approval. CI pipeline MUST fail if grep commands detect unprotected parse calls. Security review is required for any new agent implementation before merge.
</enforcement>