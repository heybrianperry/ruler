# Validate JSON Configuration Parsing in Test Suites: Integration Tests Parse

These rules are ALWAYS ACTIVE for all test suites that parse JSON configuration files or validate agent configuration outputs across the codebase's multiple AI agent platforms (Gemini CLI, Copilot, Cursor, Claude, OpenCode, Zed, Aider, Junie, Kilocode).

### Rules

- **R-JSON-001** SHOULD: Integration tests SHOULD parse and validate JSON configurations for all supported agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, Kilocode) to ensure comprehensive coverage.
- **R-JSON-002** MUST: Use JSON.parse() immediately after fs.readFile() calls to validate configuration files before asserting on properties.
- **R-JSON-003** SHOULD: Wrap JSON.parse() calls with descriptive error messages or use try-catch blocks that log file paths and agent names before re-throwing exceptions.
- **R-JSON-004** SHOULD: Structure tests with describe/it blocks organized by agent type to isolate validation failures.
- **R-JSON-005** SHOULD: For comprehensive integration tests, parse and validate JSON configurations for all agents in the expectedFiles list to ensure complete coverage.
- **R-JSON-006** SHOULD: When testing merge strategies or idempotent operations, parse JSON before and after operations to validate content stability and correctness.
- **R-JSON-007** SHOULD: Supplement JSON parsing with explicit assertions on critical configuration values like mcpServers keys, contextFileName values, and schema URLs.
- **R-JSON-008** MAY: Performance-critical tests may defer JSON parsing to reduce execution time if syntactic validation is not the primary concern.

### Verify

```bash
# Count JSON.parse usage in test files
grep -r 'JSON.parse' tests/ --include='*.test.ts' | wc -l

# Find JSON.parse combined with readFile patterns
grep -r 'JSON.parse.*readFile' tests/ --include='*.test.ts'

# Run integration and MCP-related tests with verbose output
npm test -- --testPathPattern='mcp|apply|integration' --verbose
```

**Accept when:**
- All test files that read JSON configuration files include JSON.parse() calls to validate syntactic correctness
- Integration tests successfully parse and validate JSON configurations for all supported agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, Kilocode)
- Test suites pass with clear error messages when JSON parsing fails, indicating which configuration file and agent caused the failure
- JSON.parse() calls are wrapped with descriptive error context or try-catch blocks that identify the source file and agent
- Critical configuration values (mcpServers, contextFileName, servers keys) are explicitly asserted after successful parsing

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON parsing validation in test suites. All test files reading JSON configuration files MUST include JSON.parse() validation before property assertions. Code review MUST verify that new tests reading JSON configuration files include JSON.parse() validation. CI pipeline MUST run test suites that include JSON parsing validation for all agent configurations.
</enforcement>