# Validate JSON Configuration Files Using JSON.parse in Test Suites: Tests Read Configuration

These rules are ALWAYS ACTIVE for all test suites that read and validate JSON configuration files for AI agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands, AugmentCode) and MCP server configurations.

### Rules

- **R-JSON-001** MUST: Tests MUST read configuration files using fs.readFile or fs/promises with utf8 encoding before parsing.
- **R-JSON-002** MUST: Tests MUST use JSON.parse to validate JSON configuration file syntax before making assertions on structure and content.
- **R-JSON-003** MUST: Tests MUST wrap JSON.parse calls in try-catch blocks with enhanced error messages that include file path and raw content preview.
- **R-JSON-004** MUST: Tests MUST combine JSON.parse with expect() assertions to verify both syntax validity and structural correctness (required keys, value types, nested structure).
- **R-JSON-005** MUST: Tests validating merge behavior MUST parse both source and target files and compare keys using Object.keys().sort().
- **R-JSON-006** SHOULD: Tests SHOULD use path.join(projectRoot, relativePath) to construct absolute paths to configuration files.
- **R-JSON-007** SHOULD: Tests verifying backup prevention SHOULD use fs.access() to verify .bak files do not exist after operations.
- **R-JSON-008** MAY: Tests explicitly verifying error handling for malformed JSON MAY omit JSON.parse validation if documented in test description (EXC-001).

### Verify

```bash
# Count JSON.parse usage in test files
grep -r 'JSON\.parse' tests/ --include='*.test.ts' | wc -l

# Verify fs.readFile with utf8 encoding is paired with JSON.parse
grep -r 'fs\.readFile.*utf8' tests/ --include='*.test.ts' | grep -c 'JSON\.parse'

# Run configuration validation tests for all agent integrations
npm test -- --testPathPattern='mcp|gemini|agent|copilot|cursor|claude|firebase|openhands' --passWithNoTests
```

**Accept when:**
- All test files that read JSON configuration files use JSON.parse to validate syntax before assertions
- Tests successfully detect malformed JSON by catching JSON.parse exceptions
- Configuration validation tests pass for all agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands, AugmentCode)
- fs.readFile with utf8 encoding is consistently paired with JSON.parse in configuration test files
- Error handling tests include documented exceptions for cases where JSON.parse validation is intentionally omitted

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All test files reading JSON configuration files MUST comply with R-JSON-001 through R-JSON-007. Violations MUST be caught during code review and CI pipeline execution.
</enforcement>