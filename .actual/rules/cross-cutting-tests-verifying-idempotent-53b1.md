# Validate JSON Configuration Parsing in Test Suites: Tests Verifying Idempotent

These rules are ALWAYS ACTIVE for all test suites that parse JSON configuration files or validate agent configuration outputs, including integration tests verifying MCP server configuration propagation, unit tests for agent-specific configuration generation, and tests validating merge strategies and idempotent operations on JSON configs.

### Rules

- **R-JSON-001** MUST: Tests verifying idempotent operations MUST parse JSON configurations before and after re-runs to validate content stability.
- **R-JSON-002** MUST: All test files that read JSON configuration files from the file system MUST include JSON.parse() calls to validate syntactic correctness before asserting on configuration properties.
- **R-JSON-003** MUST: JSON.parse() calls MUST be wrapped with descriptive error messages or try-catch blocks that log file paths and agent names before re-throwing exceptions.
- **R-JSON-004** SHOULD: Tests validating merge strategies or idempotent operations SHOULD parse JSON configurations for all agents in the expectedFiles list to ensure complete coverage across supported platforms (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, Kilocode).
- **R-JSON-005** SHOULD: Parsing logic SHOULD be encapsulated in agent-specific helper functions to enable independent updates when configuration formats change.
- **R-JSON-006** SHOULD: Supplement JSON parsing with explicit assertions on critical configuration values like mcpServers keys, contextFileName values, and schema URLs to validate semantic correctness beyond structure.

### Verify

```bash
# Count JSON.parse usage in test files
grep -r 'JSON.parse' tests/ --include='*.test.ts' | wc -l

# Verify JSON.parse is used with readFile patterns
grep -r 'JSON.parse.*readFile' tests/ --include='*.test.ts'

# Run integration and configuration-related tests with verbose output
npm test -- --testPathPattern='mcp|apply|integration' --verbose
```

**Accept when:**
- All test files that read JSON configuration files include JSON.parse() calls to validate syntactic correctness
- Integration tests successfully parse and validate JSON configurations for all supported agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, Kilocode)
- Test suites pass with clear error messages when JSON parsing fails, indicating which configuration file and agent caused the failure
- JSON.parse() calls are wrapped with descriptive error context (file paths, agent names) for debugging
- Tests validating idempotent operations parse JSON before and after re-runs to confirm content stability

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON parsing validation in test suites. All test files reading JSON configuration files MUST include JSON.parse() calls with proper error handling and descriptive messages. Violations will cause production issues by allowing malformed JSON configurations to pass testing.
</enforcement>