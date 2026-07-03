# Validate JSON Configuration Files Using JSON.parse in Test Suites: Tests Use Path

These rules are ALWAYS ACTIVE for all test suites that read and validate JSON configuration files for AI agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands, AugmentCode) using MCP server configurations.

### Rules

- **R-JSON-001** SHOULD: Tests SHOULD use `path.join()` to construct absolute paths to configuration files within test project roots.
- **R-JSON-002** SHOULD: Tests SHOULD use `JSON.parse()` to validate JSON configuration file syntax before making assertions on structure and content.
- **R-JSON-003** SHOULD: Tests SHOULD read configuration files with utf8 encoding using `await fs.readFile(path, 'utf8')` before parsing.
- **R-JSON-004** SHOULD: Tests SHOULD combine `JSON.parse()` with `expect()` assertions to verify both syntax validity and structural correctness of configuration objects.
- **R-JSON-005** SHOULD: Tests SHOULD wrap `JSON.parse()` calls in try-catch blocks with enhanced error messages that include file path and raw content preview.
- **R-JSON-006** MAY: Tests MAY use `Object.keys().sort()` to compare configuration keys when testing merge behavior across source and target files.

### Verify

```bash
# Count JSON.parse usage in test files
grep -r 'JSON\.parse' tests/ --include='*.test.ts' | wc -l

# Verify fs.readFile with utf8 encoding is paired with JSON.parse
grep -r 'fs\.readFile.*utf8' tests/ --include='*.test.ts' | grep -c 'JSON\.parse'

# Run configuration validation tests for all agent integrations
npm test -- --testPathPattern='mcp|gemini|agent' --passWithNoTests
```

**Accept when:**
- All test files that read JSON configuration files use `JSON.parse()` to validate syntax before assertions
- Tests successfully detect malformed JSON by catching `JSON.parse()` exceptions
- Configuration validation tests pass for all agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands)
- File paths are constructed using `path.join()` with absolute paths to test project roots
- Error handling for `JSON.parse()` exceptions includes file path and content context in error messages

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All test files validating JSON configuration files MUST comply with R-JSON-001 through R-JSON-006. Violations detected during code review or CI pipeline execution MUST be addressed before merge.
</enforcement>