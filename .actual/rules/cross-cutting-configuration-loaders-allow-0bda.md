# Validate External JSON Input with JSON.parse in Test and Agent Configuration Handlers: Configuration Loaders Allow

These rules are ALWAYS ACTIVE for all agent configuration handlers, core configuration loaders, test harnesses, and any code path that reads and processes external JSON configuration files for agent or MCP server definitions.

### Rules

- **R-CFG-001** SHOULD: Configuration loaders SHOULD allow JSON.parse to throw SyntaxError exceptions for malformed JSON, propagating parse failures to calling code for appropriate error handling.

### Verify

```bash
# Verify JSON.parse usage after fs.readFile in agent handlers and tests
grep -r 'JSON\.parse.*fs\.readFile\|JSON\.parse.*readFile' src/agents/ tests/ --include='*.ts' | wc -l

# Verify JSON.parse in core agent configuration handlers
grep -r 'JSON\.parse' src/agents/OpenCodeAgent.ts src/agents/AmazonQCliAgent.ts src/core/UnifiedConfigLoader.ts

# Verify test harnesses pass with configuration structure assertions
npm test -- --testNamePattern='(gemini-no-backup|apply-mcp|agent-specific-disable|mcp-key)' 2>&1 | grep -c 'PASS'
```

**Accept when:**
- All agent configuration handlers in src/agents/ use JSON.parse immediately after reading configuration files with fs.readFile or fs/promises readFile
- Test harnesses validate configuration structure using JSON.parse before asserting on nested properties, server definitions, or key presence
- grep verification shows consistent JSON.parse usage across at least 15 files in agent handlers and test suites

<enforcement>
Clause R-CFG-001 MUST be verified before accepting new agent configuration handlers or test harnesses that read external JSON configuration files. Code review MUST reject handlers that read JSON configuration files without JSON.parse validation. CI pipeline MUST warn when new configuration file reads are detected without corresponding JSON.parse calls.
</enforcement>