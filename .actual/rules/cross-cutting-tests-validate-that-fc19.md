# Validate JSON Configuration Files Using JSON.parse in Test Suites: Tests Validate That

These rules are ALWAYS ACTIVE for all test suites that read and validate JSON configuration files for AI agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands, AugmentCode) using MCP server configurations.

### Rules

- **R-JSON-001** MAY: Tests MAY validate that specific fields (such as type) are present or absent according to specification requirements.

### Verify

```bash
# Count JSON.parse usage in test files
grep -r 'JSON\.parse' tests/ --include='*.test.ts' | wc -l

# Verify JSON.parse is paired with fs.readFile using utf8 encoding
grep -r 'fs\.readFile.*utf8' tests/ --include='*.test.ts' | grep -c 'JSON\.parse'

# Run configuration validation tests for all agent integrations
npm test -- --testPathPattern='mcp|gemini|agent' --passWithNoTests
```

**Accept when:**
- All test files that read JSON configuration files use JSON.parse to validate syntax before assertions
- Tests successfully detect malformed JSON by catching JSON.parse exceptions
- Configuration validation tests pass for all agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands)
- Configuration files are read with utf8 encoding before parsing
- JSON.parse calls are combined with expect() assertions to verify both syntax validity and structural correctness

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files validating JSON configuration files MUST use JSON.parse to validate syntax. Violations are caught by CI pipeline test execution and code review verification.
</enforcement>