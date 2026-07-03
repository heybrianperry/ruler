# Adopt Nested MCP Configuration with Per-Directory Agent Control: Agent Specific Mcp

These rules are ALWAYS ACTIVE for all AI agent configuration systems that support MCP (Model Context Protocol) integration with nested directory structures.

### Rules

- **R-AGENT-MCP-001** MUST: Agent-specific MCP enablement flags (agents.<agent>.mcp.enabled) MUST be honored when determining whether to write MCP configuration files for that agent in a given directory.

### Verify

```bash
# Verify nested MCP configuration tests exist
grep -r 'applyAllAgentConfigs.*nested.*true' tests/integration/ || echo 'No nested MCP tests found'

# Count ruler.toml files with agent MCP configuration sections
find . -name 'ruler.toml' -exec grep -l '\[agents\..*\.mcp\]' {} \; | wc -l

# Validate MCP JSON file structure and server key presence
find . -name '.mcp.json' -o -name '.mcp.json.bak' | while read f; do jq -e '.mcpServers // .servers' "$f" > /dev/null && echo "$f: valid" || echo "$f: invalid"; done
```

**Accept when:**
- Integration tests validate that MCP configuration files are created only in directories where the corresponding agent has mcp.enabled=true in ruler.toml
- Tests confirm that MCP server definitions remain isolated to their defining directory without appearing in parent or child directory configurations
- Backup files with .bak extension are created before overwriting existing MCP configurations, and both original and backup paths are added to .gitignore

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration test failures block PR merges when nested MCP propagation behavior deviates from expected patterns. Configuration validation errors must be logged with specific directory and agent context to aid debugging.
</enforcement>