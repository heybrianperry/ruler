# Validate External JSON Input with JSON.parse in Test and Agent Configuration Handlers: Agent Implementations Validate

These rules are ALWAYS ACTIVE for all agent configuration handlers, core configuration loaders, and test harnesses that process external JSON configuration files from sources including .ruler/mcp.json, .vscode/mcp.json, .cursor/mcp.json, .gemini/settings.json, .idx/mcp.json, and .mcp.json.

### Rules

- **R-JSON-001** SHOULD: Agent implementations SHOULD validate JSON structure immediately after parsing and before accessing nested properties like mcpServers, servers, or contextFileName.
- **R-JSON-002** SHOULD: Use the pattern `const config = JSON.parse(await fs.readFile(configPath, 'utf8'))` for async file reads, or `const config = JSON.parse(fs.readFileSync(configPath, 'utf8'))` for synchronous reads.
- **R-JSON-003** SHOULD: Wrap JSON.parse calls in try-catch blocks at appropriate boundaries (e.g., agent initialization, test setup) to provide context-specific error messages that include the configuration file path.
- **R-JSON-004** SHOULD: After parsing, immediately validate the presence of expected top-level keys (e.g., mcpServers, servers, contextFileName) before accessing nested properties to provide clear error messages for structural issues.
- **R-JSON-005** SHOULD: In test harnesses, use JSON.parse validation as a precondition before assertions to ensure test failures indicate actual configuration issues rather than parse errors.

### Verify

```bash
# Count JSON.parse usage after fs.readFile operations
grep -r 'JSON\.parse.*fs\.readFile\|JSON\.parse.*readFile' src/agents/ tests/ --include='*.ts' | wc -l

# Verify JSON.parse in key agent handlers
grep -r 'JSON\.parse' src/agents/OpenCodeAgent.ts src/agents/AmazonQCliAgent.ts src/core/UnifiedConfigLoader.ts

# Run configuration-related tests
npm test -- --testNamePattern='(gemini-no-backup|apply-mcp|agent-specific-disable|mcp-key)' 2>&1 | grep -c 'PASS'
```

**Accept when:**
- All agent configuration handlers in src/agents/ use JSON.parse immediately after reading configuration files with fs.readFile or fs/promises readFile
- Test harnesses validate configuration structure using JSON.parse before asserting on nested properties, server definitions, or key presence
- grep verification shows consistent JSON.parse usage across at least 15 files in agent handlers and test suites

<enforcement>
Clause Code MUST NOT skip or defer verification. All new agent configuration handlers reading JSON files MUST include JSON.parse validation. Code review rejection is required for violations. CI pipeline warnings are triggered when new configuration file reads are detected without corresponding JSON.parse calls.
</enforcement>