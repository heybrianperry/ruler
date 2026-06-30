# Validate JSON Configuration Parsing in Test Suites: Tests Output Parsed

These rules are ALWAYS ACTIVE for all test suites that parse JSON configuration files or validate agent configuration outputs across the codebase.

### Rules

- **R-JSON-001** MAY: Tests MAY output parsed JSON content to console for manual inspection during comprehensive integration testing.
- **R-JSON-002** MUST: Tests reading JSON configuration files MUST include JSON.parse() calls to validate syntactic correctness before asserting on configuration properties.
- **R-JSON-003** SHOULD: Wrap JSON.parse() calls with descriptive error messages or try-catch blocks that log file paths and agent names before re-throwing exceptions.
- **R-JSON-004** SHOULD: Supplement JSON parsing with explicit assertions on critical configuration values like mcpServers keys, contextFileName values, and schema URLs.
- **R-JSON-005** SHOULD: Encapsulate JSON parsing logic in agent-specific helper functions that can be updated independently when configuration formats change.
- **R-JSON-006** MUST: Structure tests with describe/it blocks organized by agent type (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, Kilocode) to isolate validation failures.
- **R-JSON-007** MUST: For comprehensive integration tests, parse and validate JSON configurations for all agents in the expectedFiles list to ensure complete coverage.
- **R-JSON-008** MUST: When testing merge strategies or idempotent operations, parse JSON before and after operations to validate content stability and correctness.

### Verify

```bash
# Count JSON.parse usage in test files
grep -r 'JSON.parse' tests/ --include='*.test.ts' | wc -l

# Find JSON.parse combined with readFile patterns
grep -r 'JSON.parse.*readFile' tests/ --include='*.test.ts'

# Run test suites with verbose output for MCP, apply, and integration tests
npm test -- --testPathPattern='mcp|apply|integration' --verbose
```

**Accept when:**
- All test files that read JSON configuration files include JSON.parse() calls to validate syntactic correctness
- Integration tests successfully parse and validate JSON configurations for all supported agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, Kilocode)
- Test suites pass with clear error messages when JSON parsing fails, indicating which configuration file and agent caused the failure
- JSON parsing validation is present in integration and unit tests for agent configuration across all 6 test files

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON parsing validation in test suites. All test files reading JSON configuration files MUST include JSON.parse() calls with appropriate error handling and descriptive messages.
</enforcement>