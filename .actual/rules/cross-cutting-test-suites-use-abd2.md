# Adopt JSON.parse for External Configuration Input Validation: Test Suites Use

These rules are ALWAYS ACTIVE for all configuration loading, agent integration, external data parsing operations, and test suite validation within the Ruler CLI codebase.

### Rules

- **R-JSON-001** SHOULD: Test suites SHOULD use JSON.parse() to verify generated configuration files match expected schemas and contain required fields.
- **R-JSON-002** MUST: Always read file content as UTF-8 string using fs.readFile(path, 'utf8') before passing to JSON.parse().
- **R-JSON-003** MUST: Wrap JSON.parse() calls in try-catch blocks with error handling that logs file path and original error message.
- **R-JSON-004** MUST: Use zod schemas to validate parsed JSON structure matches expected agent configuration format.
- **R-JSON-005** MUST: Apply schema validation (zod) to parsed JSON objects before accessing nested properties in production code.
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

# Verify no eval() or Function() constructor usage for parsing
grep -r 'eval\|Function(' src/ tests/ | grep -i 'config\|json\|parse' && echo 'Found unsafe parsing' || echo 'No unsafe parsing detected'
```

**Accept when:**
- All agent configuration loading code uses JSON.parse() wrapped in try-catch blocks with error logging
- Test suites successfully parse and validate generated JSON configuration files for all supported agents
- No instances of eval() or Function() constructor used for parsing configuration data
- Schema validation (zod) is applied to parsed JSON objects before accessing nested properties in production code
- All JSON.parse() calls read file content as UTF-8 string first
- Error messages from JSON.parse() failures include file path and are surfaced to user with actionable guidance

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All JSON parsing in configuration loading and test suites MUST comply with R-JSON-001 through R-JSON-007. Violations block merge and must be remediated before acceptance.
</enforcement>