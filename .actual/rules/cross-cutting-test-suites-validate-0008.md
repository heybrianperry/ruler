# Validate JSON Configuration Files Using JSON.parse in Test Suites: Test Suites Validate

These rules are ALWAYS ACTIVE for all test suites that read and validate JSON configuration files for AI agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands, AugmentCode) using MCP server configurations.

### Rules

- **R-TSV-001** SHOULD: Test suites SHOULD validate configuration merge behavior by parsing both source and target JSON files using JSON.parse after reading with fs.readFile(path, 'utf8').
- **R-TSV-002** SHOULD: Tests SHOULD combine JSON.parse with expect() assertions to verify both syntax validity and structural correctness of configuration objects.
- **R-TSV-003** SHOULD: Tests SHOULD use path.join(projectRoot, relativePath) to construct absolute paths to configuration files before reading and parsing.
- **R-TSV-004** SHOULD: Tests verifying merge behavior SHOULD parse both source and target files and compare keys using Object.keys().sort() to validate merge correctness.
- **R-TSV-005** SHOULD: Tests SHOULD wrap JSON.parse calls in try-catch blocks with enhanced error messages that include file path and raw content preview for better diagnostics.
- **R-TSV-006** MAY: Tests MAY use fs.access() to verify .bak files do not exist after operations when testing backup prevention.

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
- fs.readFile calls with 'utf8' encoding are consistently paired with JSON.parse validation

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration test files MUST include JSON.parse validation. CI pipeline MUST fail if tests do not properly validate JSON configuration files. Code review MUST request addition of JSON.parse validation for new configuration tests unless documented exception is approved.
</enforcement>