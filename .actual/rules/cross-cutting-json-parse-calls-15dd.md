# Adopt JSON.parse for External Configuration Input Validation: Json Parse Calls

These rules are ALWAYS ACTIVE for all configuration loading, agent integration, and external data parsing operations within the Ruler CLI codebase, including agent configuration files, MCP server configuration parsing, test harness validation, and backup file restoration operations.

### Rules

- **R-JSON-001** MUST: JSON.parse() calls MUST be wrapped in try-catch blocks to handle malformed input gracefully and provide actionable error messages.
- **R-JSON-002** MUST: Always read file content as UTF-8 string using fs.readFile(path, 'utf8') before passing to JSON.parse().
- **R-JSON-003** MUST: Wrap JSON.parse() calls in try-catch blocks with error handling that logs file path and original error message.
- **R-JSON-004** MUST: Use zod schemas to validate parsed JSON structure matches expected agent configuration format immediately after parsing.
- **R-JSON-005** MUST: Validate expected top-level structure before accessing properties to prevent runtime errors from unexpected JSON types (arrays instead of objects).
- **R-JSON-006** SHOULD: In test suites, use JSON.parse() to verify generated files are valid JSON before asserting on specific fields.
- **R-JSON-007** SHOULD: For agent implementations, follow the pattern of parsing existing config, merging with new values, and writing back with JSON.stringify().

### Verify

```bash
# Count JSON.parse usage across codebase
grep -r 'JSON\.parse' src/ tests/ | wc -l

# Check for unprotected JSON.parse calls (should find none)
grep -r 'JSON\.parse' src/agents/ | grep -v 'try\|catch' && echo 'Found unprotected JSON.parse' || echo 'All JSON.parse calls protected'

# Verify JSON parsing tests pass
npm test -- --testNamePattern='should.*parse.*json|JSON' | grep -E 'PASS|FAIL'

# Verify no eval() or Function() constructor usage for parsing
grep -r 'eval\|Function(' src/ | grep -i 'json\|config\|parse' && echo 'Found unsafe parsing' || echo 'No unsafe parsing detected'
```

**Accept when:**
- All agent configuration loading code uses JSON.parse() wrapped in try-catch blocks with error logging
- Test suites successfully parse and validate generated JSON configuration files for all supported agents
- No instances of eval() or Function() constructor used for parsing configuration data
- Schema validation (zod) is applied to parsed JSON objects before accessing nested properties in production code
- All JSON.parse() calls include file path in error messages for actionable debugging
- File size checks are in place before parsing to prevent performance issues with large configuration files

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON.parse() wrapping and error handling. All configuration loading paths must be audited for compliance with R-JSON-001 through R-JSON-007 before code review approval.
</enforcement>