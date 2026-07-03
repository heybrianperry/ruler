# Adopt JSON.parse for External Configuration Input Validation: Agent Specific Configuration

These rules are ALWAYS ACTIVE for all agent configuration files, MCP server configuration parsing, test harness validation, and external data parsing operations within the Ruler CLI codebase.

### Rules

- **R-JSONPARSE-001** SHOULD: Agent-specific configuration parsers SHOULD log warnings when encountering unexpected JSON structure while continuing operation with safe defaults.
- **R-JSONPARSE-002** MUST: Always read file content as UTF-8 string using `fs.readFile(path, 'utf8')` before passing to JSON.parse().
- **R-JSONPARSE-003** MUST: Wrap all JSON.parse() calls in try-catch blocks with error handling that logs file path and original error message.
- **R-JSONPARSE-004** MUST: Use zod schemas to validate parsed JSON structure matches expected agent configuration format before accessing nested properties.
- **R-JSONPARSE-005** MUST: In test suites, use JSON.parse() to verify generated files are valid JSON before asserting on specific fields.
- **R-JSONPARSE-006** MUST: For agent implementations, follow the pattern of parsing existing config, merging with new values, and writing back with JSON.stringify().
- **R-JSONPARSE-007** MUST NOT: Use eval() or Function() constructor for parsing configuration data.

### Verify

```bash
# Count JSON.parse usage across codebase
grep -r 'JSON\.parse' src/ tests/ | wc -l

# Verify all JSON.parse calls are protected with try-catch
grep -r 'JSON\.parse' src/agents/ | grep -v 'try\|catch' && echo 'Found unprotected JSON.parse' || echo 'All JSON.parse calls protected'

# Run JSON parsing related tests
npm test -- --testNamePattern='should.*parse.*json|JSON' | grep -E 'PASS|FAIL'

# Check for eval() usage in configuration code
grep -r 'eval(' src/ tests/ && echo 'Found eval() usage' || echo 'No eval() usage detected'
```

**Accept when:**
- All agent configuration loading code uses JSON.parse() wrapped in try-catch blocks with error logging
- Test suites successfully parse and validate generated JSON configuration files for all supported agents
- No instances of eval() or Function() constructor used for parsing configuration data
- Schema validation (zod) is applied to parsed JSON objects before accessing nested properties in production code
- All agent-specific configuration parsers log warnings for unexpected JSON structure while continuing with safe defaults
- File I/O operations read configuration files as UTF-8 strings before parsing

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON.parse() usage patterns. All configuration loading paths must be validated against these rules before code is committed.
</enforcement>