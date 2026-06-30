# Validate JSON Configuration Parsing in Test Suites: Test Suites Use

These rules are ALWAYS ACTIVE for all test suites that parse JSON configuration files or validate agent configuration outputs across the codebase.

### Rules

- **R-JSON-001** SHOULD: Test suites SHOULD use describe/it structure from testing frameworks to organize JSON validation tests by agent type and configuration scenario.
- **R-JSON-002** MUST: Use JSON.parse() immediately after fs.readFile() calls to validate configuration files before asserting on properties.
- **R-JSON-003** MUST: Wrap JSON.parse() calls with descriptive error messages or use try-catch blocks that log file paths and agent names before re-throwing exceptions.
- **R-JSON-004** SHOULD: Structure tests with describe/it blocks organized by agent type (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, Kilocode) to isolate validation failures.
- **R-JSON-005** SHOULD: Supplement JSON parsing with explicit assertions on critical configuration values like mcpServers keys, contextFileName values, and schema URLs.
- **R-JSON-006** MUST: For comprehensive integration tests, parse and validate JSON configurations for all agents in the expectedFiles list to ensure complete coverage.
- **R-JSON-007** SHOULD: When testing merge strategies or idempotent operations, parse JSON before and after operations to validate content stability and correctness.
- **R-JSON-008** MAY: Encapsulate parsing logic in agent-specific helper functions that can be updated independently when configuration formats change.

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
- JSON parsing validation is organized by agent type using describe/it blocks
- Critical configuration values (mcpServers, contextFileName, servers keys) are explicitly asserted after parsing

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON parsing validation in test suites. All test files reading JSON configuration files MUST include JSON.parse() calls with descriptive error handling. Code review MUST verify that new tests reading JSON configuration files include JSON.parse() validation before merge.
</enforcement>