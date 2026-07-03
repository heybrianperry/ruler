# Adopt Nested MCP Configuration with Per-Directory Agent Control: Mcp Configuration Files

These rules are ALWAYS ACTIVE for all AI agent configuration systems that support MCP (Model Context Protocol) integration with nested directory structures, including projects with per-agent MCP server configurations distributed across project root, module, and submodule directories.

### Rules

- **R-MCP-001** MUST: MCP configuration files MUST be written to the directory level where the corresponding .ruler/ruler.toml file defines agent and MCP server settings.
- **R-MCP-002** MUST: Agent-specific MCP enablement flags (e.g., agents.copilot.mcp.enabled) defined in a directory's ruler.toml MUST be honored without inheritance from parent directories.
- **R-MCP-003** MUST: MCP server definitions (stdio commands and remote URLs) MUST remain scoped to their defining directory without propagating to parent or child directory configurations.
- **R-MCP-004** MUST: Backup files with .bak extension MUST be created before overwriting existing MCP configurations.
- **R-MCP-005** MUST: Both MCP configuration file paths and their .bak backup paths MUST be added to .gitignore using relative paths from project root.
- **R-MCP-006** SHOULD: Use the applyAllAgentConfigs function with nested=true flag to enable per-directory MCP configuration mode.
- **R-MCP-007** SHOULD: Implement configuration validation tooling that reports all active agents and MCP servers across the directory tree.
- **R-MCP-008** MAY: Legacy projects migrating to nested MCP may temporarily disable backup creation if existing configurations are already version-controlled (EXC-002).

### Verify

```bash
# Verify nested MCP configuration tests are present
grep -r 'applyAllAgentConfigs.*nested.*true' tests/integration/ || echo 'No nested MCP tests found'

# Count ruler.toml files with MCP agent configuration sections
find . -name 'ruler.toml' -exec grep -l '\[agents\..*\.mcp\]' {} \; | wc -l

# Validate MCP JSON file structure and server key presence
find . -name '.mcp.json' -o -name '.mcp.json.bak' | while read f; do jq -e '.mcpServers // .servers' "$f" > /dev/null && echo "$f: valid" || echo "$f: invalid"; done
```

**Accept when:**
- Integration tests validate that MCP configuration files are created only in directories where the corresponding agent has mcp.enabled=true in ruler.toml
- Tests confirm that MCP server definitions remain isolated to their defining directory without appearing in parent or child directory configurations
- Backup files with .bak extension are created before overwriting existing MCP configurations, and both original and backup paths are added to .gitignore
- Configuration validation errors are logged with specific directory and agent context
- File system assertions verify expected MCP config presence/absence based on agent enablement flags

<enforcement>
Claude Code MUST NOT skip or defer verification of nested MCP configuration isolation. Integration test failures blocking PR merges when nested MCP propagation behavior deviates from expected patterns are mandatory enforcement gates.
</enforcement>