# Validate JSON Configuration Files Using JSON.parse in Test Suites: Tests Verify That

These rules are ALWAYS ACTIVE for all test suites that read and validate JSON configuration files for AI agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands, AugmentCode) using MCP server configurations.

### Rules

- **R-JSON-001** SHOULD: Tests SHOULD verify that backup files (.bak) are not created during idempotent configuration operations.
- **R-JSON-002** MUST: Tests that read JSON configuration files MUST use JSON.parse to validate syntax before making assertions.
- **R-JSON-003** MUST: Configuration file reads MUST use utf8 encoding: `await fs.readFile(path, 'utf8')` before parsing.
- **R-JSON-004** MUST: Tests MUST combine JSON.parse with expect() assertions to verify both syntax validity and structural correctness.
- **R-JSON-005** SHOULD: Tests verifying merge behavior SHOULD parse both source and target files to compare keys using Object.keys().sort().
- **R-JSON-006** SHOULD: Tests SHOULD wrap JSON.parse calls in try-catch blocks with enhanced error messages that include file path and raw content preview.
- **R-JSON-007** SHOULD: Tests SHOULD use path.join(projectRoot, relativePath) to construct absolute paths to configuration files.
- **R-JSON-008** MAY: Tests MAY document exceptions to JSON.parse validation in test comments with rationale when testing error handling (EXC-001).

### Verify

```bash
# Count JSON.parse usage in test files
grep -r 'JSON\.parse' tests/ --include='*.test.ts' | wc -l

# Verify fs.readFile with utf8 encoding is paired with JSON.parse
grep -r 'fs\.readFile.*utf8' tests/ --include='*.test.ts' | grep -c 'JSON\.parse'

# Run configuration validation tests for all agent integrations
npm test -- --testPathPattern='mcp|gemini|agent' --passWithNoTests

# Verify no .bak files are created during test execution
find tests/ -name '*.bak' -type f | wc -l
```

**Accept when:**
- All test files that read JSON configuration files use JSON.parse to validate syntax before assertions
- Tests successfully detect malformed JSON by catching JSON.parse exceptions
- Configuration validation tests pass for all agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands)
- No backup (.bak) files are created during idempotent configuration test operations
- JSON.parse calls are paired with utf8 encoding in file reads
- Assertions on configuration structure follow JSON.parse validation

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files validating JSON configuration files MUST comply with R-JSON-002 and R-JSON-003. Violations detected in CI pipeline MUST be addressed before merge.
</enforcement>