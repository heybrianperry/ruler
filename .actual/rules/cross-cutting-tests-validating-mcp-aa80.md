# Validate JSON Configuration Parsing in Test Suites: Tests Validating Mcp

These rules are ALWAYS ACTIVE for all test suites that parse JSON configuration files or validate agent configuration outputs, including integration tests verifying MCP server configuration propagation, unit tests for agent-specific configuration generation, and tests validating merge strategies for MCP configurations.

### Rules

- **R-MCP-001** MUST: Tests validating MCP server configurations MUST parse the configuration JSON and verify the presence of configured server entries (filesystem_server, remote_api) within the appropriate key structure.

### Verify

```bash
# Count JSON.parse usage in test files
grep -r 'JSON.parse' tests/ --include='*.test.ts' | wc -l

# Find JSON.parse calls combined with readFile operations
grep -r 'JSON.parse.*readFile' tests/ --include='*.test.ts'

# Run MCP, apply, and integration test suites with verbose output
npm test -- --testPathPattern='mcp|apply|integration' --verbose
```

**Accept when:**
- All test files that read JSON configuration files include JSON.parse() calls to validate syntactic correctness
- Integration tests successfully parse and validate JSON configurations for all supported agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, Kilocode)
- Test suites pass with clear error messages when JSON parsing fails, indicating which configuration file and agent caused the failure
- JSON.parse() calls are wrapped with descriptive error messages or try-catch blocks that log file paths and agent names
- Tests include explicit assertions on critical configuration values like mcpServers keys, contextFileName values, and schema URLs

<enforcement>
Clause Code MUST NOT skip or defer verification of JSON parsing validation in test suites. All test files reading JSON configuration files MUST include JSON.parse() validation before asserting on configuration properties. Violations will be caught during code review and CI pipeline execution.
</enforcement>