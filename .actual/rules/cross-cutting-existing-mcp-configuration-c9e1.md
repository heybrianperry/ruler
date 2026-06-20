# Adopt Nested MCP Configuration with Per-Directory Agent Control: Existing Mcp Configuration

These rules are ALWAYS ACTIVE for all AI agent configuration systems that support MCP (Model Context Protocol) integration with nested directory structures, including projects with per-directory .ruler/ruler.toml files defining agent enablement and MCP server definitions across project root, module, and submodule directories.

### Rules

- **R-MCP-001** MUST: Existing MCP configuration files MUST be backed up with a .bak extension before being overwritten.

### Verify

```bash
# Verify nested MCP configuration test coverage
grep -r 'applyAllAgentConfigs.*nested.*true' tests/integration/ || echo 'No nested MCP tests found'

# Count ruler.toml files with MCP agent sections
find . -name 'ruler.toml' -exec grep -l '\[agents\..*\.mcp\]' {} \; | wc -l

# Validate MCP JSON files have correct structure and backup paths
find . -name '.mcp.json' -o -name '.mcp.json.bak' | while read f; do jq -e '.mcpServers // .servers' "$f" > /dev/null && echo "$f: valid" || echo "$f: invalid"; done
```

**Accept when:**
- Integration tests validate that MCP configuration files are created only in directories where the corresponding agent has mcp.enabled=true in ruler.toml
- Tests confirm that MCP server definitions remain isolated to their defining directory without appearing in parent or child directory configurations
- Backup files with .bak extension are created before overwriting existing MCP configurations, and both original and backup paths are added to .gitignore

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration test failures block PR merges when backup creation or nested MCP propagation behavior deviates from expected patterns. Configuration validation errors must be logged with specific directory and agent context.
</enforcement>