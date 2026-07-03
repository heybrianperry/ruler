# Validate JSON Configuration Parsing in Test Suites: Test Suites Parse

These rules are ALWAYS ACTIVE for all test suites that parse JSON configuration files or validate agent configuration outputs across the codebase.

### Rules

- **R-TSP-001** MUST: Test suites MUST parse JSON configuration files using JSON.parse() after reading file content to validate syntactic correctness before asserting on configuration properties.

### Verify

```bash
# Count JSON.parse usage in test files
grep -r 'JSON.parse' tests/ --include='*.test.ts' | wc -l

# Verify JSON.parse is used with readFile patterns
grep -r 'JSON.parse.*readFile' tests/ --include='*.test.ts'

# Run test suites with verbose output
npm test -- --testPathPattern='mcp|apply|integration' --verbose
```

**Accept when:**
- All test files that read JSON configuration files include JSON.parse() calls to validate syntactic correctness
- Integration tests successfully parse and validate JSON configurations for all supported agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, Kilocode)
- Test suites pass with clear error messages when JSON parsing fails, indicating which configuration file and agent caused the failure
- JSON parsing validation is present in tests for MCP backup prevention, empty server key fixes, comprehensive integration workflows, merge strategies, and idempotent operations

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON.parse() validation in test suites. All test files reading JSON configuration outputs from ruler apply commands or agent-specific settings must include explicit JSON parsing validation before property assertions.
</enforcement>