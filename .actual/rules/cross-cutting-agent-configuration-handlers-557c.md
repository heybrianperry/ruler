# Validate External JSON Input with JSON.parse in Test and Agent Configuration Handlers: Agent Configuration Handlers

These rules are ALWAYS ACTIVE for all agent configuration handlers and test harnesses that process external JSON configuration files from .ruler/mcp.json, .vscode/mcp.json, .cursor/mcp.json, .gemini/settings.json, .idx/mcp.json, and .mcp.json sources.

### Rules

- **R-AGENT-CONFIG-001** MUST: All agent configuration handlers MUST use JSON.parse to validate external JSON configuration files immediately after reading file content with fs.readFile or fs/promises readFile.

### Verify

```bash
# Count JSON.parse usage after fs.readFile in agent and test files
grep -r 'JSON\.parse.*fs\.readFile\|JSON\.parse.*readFile' src/agents/ tests/ --include='*.ts' | wc -l

# Verify JSON.parse usage in core agent handlers
grep -r 'JSON\.parse' src/agents/OpenCodeAgent.ts src/agents/AmazonQCliAgent.ts src/core/UnifiedConfigLoader.ts

# Run configuration-related tests
npm test -- --testNamePattern='(gemini-no-backup|apply-mcp|agent-specific-disable|mcp-key)' 2>&1 | grep -c 'PASS'
```

**Accept when:**
- All agent configuration handlers in src/agents/ use JSON.parse immediately after reading configuration files with fs.readFile or fs/promises readFile
- Test harnesses validate configuration structure using JSON.parse before asserting on nested properties, server definitions, or key presence
- grep verification shows consistent JSON.parse usage across at least 15 files in agent handlers and test suites

<enforcement>
Clause R-AGENT-CONFIG-001 MUST be verified before accepting new agent configuration handlers or test harnesses that read external JSON files. Code review rejection is mandatory for violations. Claude Code MUST NOT skip or defer verification.
</enforcement>