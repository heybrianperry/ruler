# Adopt JSON.parse for External Configuration Input Validation: Parsed Json Objects

These rules are ALWAYS ACTIVE for all agent configuration files, MCP server configuration parsing, test harness validation, VSCode settings integration, and backup file restoration operations within the Ruler CLI codebase.

### Rules

- **R-JSON-001** MUST: Parsed JSON objects MUST be validated against expected schemas (using zod or type guards) before accessing nested properties.
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

# Check for unprotected JSON.parse calls (should find none)
grep -r 'JSON\.parse' src/agents/ | grep -v 'try\|catch' && echo 'Found unprotected JSON.parse' || echo 'All JSON.parse calls protected'

# Verify JSON parsing tests pass
npm test -- --testNamePattern='should.*parse.*json|JSON' | grep -E 'PASS|FAIL'

# Check for eval() usage (should find none)
grep -r 'eval(' src/ tests/ && echo 'Found eval() usage' || echo 'No eval() usage detected'
```

**Accept when:**
- All agent configuration loading code uses JSON.parse() wrapped in try-catch blocks with error logging
- Test suites successfully parse and validate generated JSON configuration files for all supported agents
- No instances of eval() or Function() constructor used for parsing configuration data
- Schema validation (zod) is applied to parsed JSON objects before accessing nested properties in production code
- All agent configuration files (.mcp.json, settings.json, opencode.json, .gemini/settings.json, .qwen/settings.json, .zed/settings.json, etc.) are validated against expected schemas
- VSCode settings.json parsing for AugmentCode MCP server integration includes schema validation
- Backup file restoration operations that read .bak JSON files include error handling and validation

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON parsing rules. All JSON.parse() calls must be protected with try-catch blocks and schema validation before merge.
</enforcement>