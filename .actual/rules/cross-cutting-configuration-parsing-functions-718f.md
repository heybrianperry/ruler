# Validate JSON Input from File System Before Parsing in Security-Critical Contexts: Configuration Parsing Functions

These rules are ALWAYS ACTIVE for all code that reads configuration files from the file system and parses them as JSON or TOML, particularly in agent configuration management and MCP server handling across .gemini/, .vscode/, .cursor/, .idx/, .ruler/ directories and test harness code.

### Rules

- **R-CFG-001** SHOULD: Configuration parsing functions SHOULD return typed objects or throw descriptive errors rather than returning raw parsed data.

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
- Parsed configuration objects are validated for expected structure (e.g., mcpServers, servers, contextFileName) before use in agent configuration or MCP server definitions
- Test suite includes cases for malformed JSON/TOML files that verify proper error handling and descriptive error messages
- Error messages are sanitized to remove absolute paths and sensitive configuration details
- Shared utility functions like safeParseJSON(content: string, filePath: string) and safeParseConfig(filePath: string) are used to centralize error handling patterns

<enforcement>
Clause Code MUST NOT skip or defer verification. All JSON.parse() and parseTOML() calls must be protected by error handling. Code review MUST block merge if configuration parsing lacks error handling. CI pipeline MUST fail if grep commands detect unprotected parse calls. Security review is REQUIRED for any new agent implementation before merge.
</enforcement>