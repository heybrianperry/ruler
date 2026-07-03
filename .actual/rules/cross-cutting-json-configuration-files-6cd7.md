# Adopt JSON.parse for External Configuration Input Validation: Json Configuration Files

These rules are ALWAYS ACTIVE for all configuration loading, agent integration, and external data parsing operations within the Ruler CLI codebase, including agent configuration files, MCP server configuration parsing, test harness validation, and backup file restoration operations.

### Rules

- **R-JSON-001** MUST: All JSON configuration files read from the filesystem MUST be parsed using JSON.parse() to validate structure before processing.
- **R-JSON-002** MUST: All JSON.parse() calls MUST be wrapped in try-catch blocks with error handling that logs file path and original error message.
- **R-JSON-003** MUST: File content MUST be read as UTF-8 string using fs.readFile(path, 'utf8') before passing to JSON.parse().
- **R-JSON-004** MUST: Schema validation using zod MUST be applied to parsed JSON objects before accessing nested properties in production code.
- **R-JSON-005** MUST: No instances of eval() or Function() constructor MUST be used for parsing configuration data.
- **R-JSON-006** SHOULD: In test suites, use JSON.parse() to verify generated files are valid JSON before asserting on specific fields.
- **R-JSON-007** SHOULD: For agent implementations, follow the pattern of parsing existing config, merging with new values, and writing back with JSON.stringify().

### Verify

```bash
# Count JSON.parse usage across codebase
grep -r 'JSON\.parse' src/ tests/ | wc -l

# Verify all JSON.parse calls are protected with try-catch
grep -r 'JSON\.parse' src/agents/ | grep -v 'try\|catch' && echo 'Found unprotected JSON.parse' || echo 'All JSON.parse calls protected'

# Run JSON parsing related tests
npm test -- --testNamePattern='should.*parse.*json|JSON' | grep -E 'PASS|FAIL'

# Verify no eval() usage for configuration parsing
grep -r 'eval(' src/ tests/ && echo 'Found eval() usage' || echo 'No eval() usage detected'
```

**Accept when:**
- All agent configuration loading code uses JSON.parse() wrapped in try-catch blocks with error logging
- Test suites successfully parse and validate generated JSON configuration files for all supported agents
- No instances of eval() or Function() constructor used for parsing configuration data
- Schema validation (zod) is applied to parsed JSON objects before accessing nested properties in production code
- All JSON configuration files are read as UTF-8 strings before parsing
- Error messages from JSON.parse() failures include file path and are surfaced to user with actionable guidance

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All JSON configuration parsing MUST use JSON.parse() with proper error handling and schema validation before merging into production.
</enforcement>