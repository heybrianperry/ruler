# Validate External JSON Input with JSON.parse in Test and Agent Configuration Handlers: Configuration Handlers Combine

These rules are ALWAYS ACTIVE for all agent configuration handlers, test harnesses, and any code path that reads and processes external JSON configuration files for agent or MCP server definitions.

### Rules

- **R-CFG-001** MAY: Configuration handlers MAY combine JSON.parse with additional schema validation for complex configuration objects containing server definitions, command arrays, or URL specifications.

### Verify

```bash
# Verify JSON.parse usage after fs.readFile in agent handlers and tests
grep -r 'JSON\.parse.*fs\.readFile\|JSON\.parse.*readFile' src/agents/ tests/ --include='*.ts' | wc -l

# Verify specific agent configuration handlers use JSON.parse
grep -r 'JSON\.parse' src/agents/OpenCodeAgent.ts src/agents/AmazonQCliAgent.ts src/core/UnifiedConfigLoader.ts

# Verify test harnesses pass with configuration structure assertions
npm test -- --testNamePattern='(gemini-no-backup|apply-mcp|agent-specific-disable|mcp-key)' 2>&1 | grep -c 'PASS'
```

**Accept when:**
- All agent configuration handlers in src/agents/ use JSON.parse immediately after reading configuration files with fs.readFile or fs/promises readFile
- Test harnesses validate configuration structure using JSON.parse before asserting on nested properties, server definitions, or key presence
- grep verification shows consistent JSON.parse usage across at least 15 files in agent handlers and test suites

<enforcement>
Clause Code MUST NOT skip or defer verification. All new agent configuration handlers reading JSON files MUST use JSON.parse for immediate structural validation. Code review rejection is mandatory for violations.
</enforcement>