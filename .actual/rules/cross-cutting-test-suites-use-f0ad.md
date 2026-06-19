# Validate JSON Configuration Files Using JSON.parse in Test Suites: Test Suites Use

These rules are ALWAYS ACTIVE for all test suites that read and validate JSON configuration files for AI agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands, AugmentCode) and MCP server configurations.

### Rules

- **R-JSON-001** MUST: Test suites MUST use JSON.parse to validate JSON configuration files read from the filesystem before making assertions on their content.
- **R-JSON-002** MUST: Always read configuration files with utf8 encoding using `await fs.readFile(path, 'utf8')` before parsing.
- **R-JSON-003** MUST: Use `path.join(projectRoot, relativePath)` to construct absolute paths to configuration files in test projects.
- **R-JSON-004** MUST: Combine JSON.parse with `expect()` assertions to verify both syntax validity and structural correctness.
- **R-JSON-005** SHOULD: Wrap JSON.parse calls in try-catch blocks with enhanced error messages that include file path and raw content preview.
- **R-JSON-006** SHOULD: For tests verifying merge behavior, parse both source and target files to compare keys using `Object.keys().sort()`.
- **R-JSON-007** SHOULD: When testing backup prevention, use `fs.access()` to verify `.bak` files do not exist after operations.
- **R-JSON-008** MAY: Tests explicitly verifying error handling for malformed JSON may omit JSON.parse validation (EXC-001).

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
- All test files that read JSON configuration files use JSON.parse to validate syntax before assertions
- Tests successfully detect malformed JSON by catching JSON.parse exceptions
- Configuration validation tests pass for all agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands)
- File reads use utf8 encoding and are paired with JSON.parse calls
- Error messages from JSON.parse failures include file path context

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files reading JSON configuration files MUST include JSON.parse validation before assertions. Violations must be caught during code review and CI pipeline execution.
</enforcement>