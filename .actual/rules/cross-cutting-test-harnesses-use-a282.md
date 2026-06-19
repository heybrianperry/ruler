# Validate External JSON Input with JSON.parse in Test and Agent Configuration Handlers: Test Harnesses Use

These rules are ALWAYS ACTIVE for all agent configuration handlers and test harnesses that process external JSON configuration files from .ruler/mcp.json, .vscode/mcp.json, .cursor/mcp.json, .gemini/settings.json, .idx/mcp.json, and .mcp.json sources.

### Rules

- **R-JSON-001** MUST: Test harnesses MUST use JSON.parse to validate configuration file contents before asserting on configuration structure, server definitions, or key presence.
- **R-JSON-002** MUST: Agent configuration handlers (OpenCodeAgent, MistralVibeAgent, AmazonQCliAgent) MUST use JSON.parse immediately after reading configuration files with fs.readFile or fs/promises readFile.
- **R-JSON-003** MUST: Wrap JSON.parse calls in try-catch blocks at appropriate boundaries (e.g., agent initialization, test setup) to provide context-specific error messages that include the configuration file path.
- **R-JSON-004** MUST: After parsing, immediately validate the presence of expected top-level keys (e.g., mcpServers, servers, contextFileName) before accessing nested properties to provide clear error messages for structural issues.
- **R-JSON-005** SHOULD: Use the pattern `const config = JSON.parse(await fs.readFile(configPath, 'utf8'))` for async file reads, or `const config = JSON.parse(fs.readFileSync(configPath, 'utf8'))` for synchronous reads.

### Verify

```bash
# Count JSON.parse usage after fs.readFile operations
grep -r 'JSON\.parse.*fs\.readFile\|JSON\.parse.*readFile' src/agents/ tests/ --include='*.ts' | wc -l

# Verify specific agent handlers use JSON.parse
grep -r 'JSON\.parse' src/agents/OpenCodeAgent.ts src/agents/AmazonQCliAgent.ts src/core/UnifiedConfigLoader.ts

# Run configuration-related tests
npm test -- --testNamePattern='(gemini-no-backup|apply-mcp|agent-specific-disable|mcp-key)' 2>&1 | grep -c 'PASS'
```

**Accept when:**
- All agent configuration handlers in src/agents/ use JSON.parse immediately after reading configuration files with fs.readFile or fs/promises readFile
- Test harnesses validate configuration structure using JSON.parse before asserting on nested properties, server definitions, or key presence
- grep verification shows consistent JSON.parse usage across at least 15 files in agent handlers and test suites
- Configuration parsing tests pass with expected structure assertions

<enforcement>
Clause R-JSON-001 through R-JSON-005 are mandatory. Code review MUST reject new agent configuration handlers that read JSON configuration files without JSON.parse validation. CI pipeline MUST warn when new configuration file reads are detected without corresponding JSON.parse calls. Claude Code MUST NOT skip or defer verification of these rules.
</enforcement>