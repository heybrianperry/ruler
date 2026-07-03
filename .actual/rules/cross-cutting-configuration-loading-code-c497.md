# Adopt JSON.parse for External Configuration Input Validation: Configuration Loading Code

These rules are ALWAYS ACTIVE for all agent configuration files, MCP server configuration parsing, test harness validation of generated configuration outputs, VSCode settings.json parsing, and backup file restoration operations that read JSON files.

### Rules

- **R-CONFIG-001** MUST NOT: Configuration loading code MUST NOT use eval() or other dynamic code execution methods to parse JSON data.
- **R-CONFIG-002** MUST: Always read file content as UTF-8 string using fs.readFile(path, 'utf8') before passing to JSON.parse().
- **R-CONFIG-003** MUST: Wrap JSON.parse() calls in try-catch blocks with error handling that logs file path and original error message.
- **R-CONFIG-004** MUST: Use zod schemas to validate parsed JSON structure matches expected agent configuration format before accessing nested properties.
- **R-CONFIG-005** SHOULD: In test suites, use JSON.parse() to verify generated files are valid JSON before asserting on specific fields.
- **R-CONFIG-006** SHOULD: For agent implementations, follow the pattern of parsing existing config, merging with new values, and writing back with JSON.stringify().

### Verify

```bash
# Count JSON.parse usage across codebase
grep -r 'JSON\.parse' src/ tests/ | wc -l

# Verify all JSON.parse calls are protected with try-catch
grep -r 'JSON\.parse' src/agents/ | grep -v 'try\|catch' && echo 'Found unprotected JSON.parse' || echo 'All JSON.parse calls protected'

# Verify JSON parsing tests pass
npm test -- --testNamePattern='should.*parse.*json|JSON' | grep -E 'PASS|FAIL'

# Verify no eval() usage in configuration loading
grep -r 'eval(' src/ tests/ && echo 'Found eval() usage' || echo 'No eval() usage detected'
```

**Accept when:**
- All agent configuration loading code uses JSON.parse() wrapped in try-catch blocks with error logging
- Test suites successfully parse and validate generated JSON configuration files for all supported agents
- No instances of eval() or Function() constructor used for parsing configuration data
- Schema validation (zod) is applied to parsed JSON objects before accessing nested properties in production code
- All configuration files are read as UTF-8 strings before parsing
- Error messages include file path and original error details

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All JSON parsing in configuration loading must comply with R-CONFIG-001 through R-CONFIG-006 before code review approval.
</enforcement>