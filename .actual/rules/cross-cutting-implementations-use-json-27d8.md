# Adopt JSON.parse for External Configuration Input Validation: Implementations Use Json

These rules are ALWAYS ACTIVE for all configuration loading, agent integration, and external data parsing operations within the Ruler CLI codebase, including agent configuration files, MCP server configuration parsing, test harness validation, and backup file restoration operations.

### Rules

- **R-JSON-001** MUST: Use JSON.parse() wrapped in try-catch blocks for all external configuration input validation with error logging that includes file path and original error message.
- **R-JSON-002** MUST: Apply schema validation using zod immediately after JSON.parse() to validate parsed JSON structure matches expected agent configuration format before accessing nested properties.
- **R-JSON-003** MUST: Read file content as UTF-8 string using fs.readFile(path, 'utf8') before passing to JSON.parse().
- **R-JSON-004** MUST: Not use eval() or Function() constructor for parsing configuration data; use JSON.parse() exclusively.
- **R-JSON-005** MAY: Use JSON.stringify() with formatting options when writing configuration files to maintain readability.
- **R-JSON-006** SHOULD: Add file size checks before parsing to prevent performance issues or stack overflow in JSON.parse() for large configuration files.
- **R-JSON-007** SHOULD: Implement pre-flight validation command and clear error messages suggesting JSON validation tools for user-edited configuration files.

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
- File size checks are implemented before parsing large configuration files
- Error messages from JSON.parse() failures include file path and are surfaced to user with actionable guidance

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All JSON parsing operations must be validated before merge.
</enforcement>