# Validate JSON Input from File System Before Parsing in Security-Critical Contexts: Parsed Configuration Objects

These rules are ALWAYS ACTIVE for all code that reads configuration files from the file system and parses them as JSON or TOML, particularly in agent configuration management and MCP server handling.

### Rules

- **R-CFG-001** MUST: Parsed configuration objects MUST be validated for expected structure (presence of required keys like mcpServers, servers, contextFileName) before use.

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
- Error messages are sanitized to remove absolute paths and sensitive configuration details

<enforcement>
Clause Code MUST NOT skip or defer verification. All file system parsing operations require error handling and structural validation before use in security-critical contexts.
</enforcement>