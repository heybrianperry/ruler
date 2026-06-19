# Validate JSON Configuration Files Using JSON.parse in Test Suites: Tests Verify Presence

These rules are ALWAYS ACTIVE for all test suites that read and validate JSON configuration files for AI agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands, AugmentCode) using MCP server configurations.

### Rules

- **R-JSON-001** MUST: Tests MUST verify the presence and structure of agent-specific configuration keys (servers, mcpServers, contextFileName) after parsing JSON configuration files.
- **R-JSON-002** MUST: Configuration files MUST be read with utf8 encoding using `await fs.readFile(path, 'utf8')` before JSON.parse validation.
- **R-JSON-003** MUST: JSON.parse calls MUST be wrapped in try-catch blocks with enhanced error messages that include file path and raw content preview.
- **R-JSON-004** SHOULD: Tests SHOULD combine JSON.parse with explicit expect() assertions to verify both syntax validity and structural correctness of configuration objects.
- **R-JSON-005** SHOULD: For tests verifying merge behavior, both source and target files SHOULD be parsed and compared using Object.keys().sort() to validate key consistency.
- **R-JSON-006** MAY: Tests MAY explicitly verify error handling for malformed JSON as documented exceptions (EXC-001).

### Verify

```bash
# Count JSON.parse usage in test files
grep -r 'JSON\.parse' tests/ --include='*.test.ts' | wc -l

# Verify JSON.parse is paired with fs.readFile utf8 encoding
grep -r 'fs\.readFile.*utf8' tests/ --include='*.test.ts' | grep -c 'JSON\.parse'

# Run configuration validation tests for all agent integrations
npm test -- --testPathPattern='mcp|gemini|agent' --passWithNoTests
```

**Accept when:**
- All test files that read JSON configuration files use JSON.parse to validate syntax before assertions
- Tests successfully detect malformed JSON by catching JSON.parse exceptions
- Configuration validation tests pass for all agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands)
- JSON.parse calls are wrapped in try-catch blocks with descriptive error messages
- Tests verify presence of agent-specific keys (servers, mcpServers, contextFileName) after parsing

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON.parse usage in configuration test suites. All configuration tests MUST include explicit JSON parsing and structural validation before assertions.
</enforcement>