# Validate JSON Configuration Parsing in Test Suites: Tests That Verify

These rules are ALWAYS ACTIVE for all test suites that parse JSON configuration files or validate agent configuration outputs across the codebase's multiple AI agent platforms (Gemini CLI, Copilot, Cursor, Claude, OpenCode, Zed, Aider, Junie, Kilocode).

### Rules

- **R-JSON-001** MUST: Tests that verify agent configuration outputs MUST validate the presence of expected JSON keys (mcpServers, servers, contextFileName, $schema) after successful parsing.
- **R-JSON-002** MUST: Use JSON.parse() immediately after fs.readFile() calls to validate configuration files before asserting on properties.
- **R-JSON-003** MUST: Wrap JSON.parse() calls with descriptive error messages or use try-catch blocks that log file paths and agent names before re-throwing exceptions.
- **R-JSON-004** SHOULD: Structure tests with describe/it blocks organized by agent type (Gemini CLI, Copilot, Cursor, etc.) to isolate validation failures.
- **R-JSON-005** SHOULD: For comprehensive integration tests, parse and validate JSON configurations for all agents in the expectedFiles list to ensure complete coverage.
- **R-JSON-006** SHOULD: When testing merge strategies or idempotent operations, parse JSON before and after operations to validate content stability and correctness.
- **R-JSON-007** MAY: Performance-critical tests may defer JSON parsing to reduce execution time if syntactic validation is not the primary concern, with documented justification.

### Verify

```bash
# Count JSON.parse usage in test files
grep -r 'JSON.parse' tests/ --include='*.test.ts' | wc -l

# Find JSON.parse calls combined with readFile
grep -r 'JSON.parse.*readFile' tests/ --include='*.test.ts'

# Run configuration and integration tests with verbose output
npm test -- --testPathPattern='mcp|apply|integration' --verbose
```

**Accept when:**
- All test files that read JSON configuration files include JSON.parse() calls to validate syntactic correctness
- Integration tests successfully parse and validate JSON configurations for all supported agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, Kilocode)
- Test suites pass with clear error messages when JSON parsing fails, indicating which configuration file and agent caused the failure
- JSON.parse() calls are wrapped with descriptive error context (file paths, agent names) for debugging
- Expected JSON keys (mcpServers, servers, contextFileName, $schema) are explicitly validated after parsing succeeds

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON parsing validation in test suites. All test files reading JSON configuration outputs MUST include JSON.parse() validation with explicit key presence checks before asserting on configuration properties. Violations will be caught by CI pipeline test execution and code review.
</enforcement>