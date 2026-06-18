# Adopt Nested MCP Configuration with Per-Directory Agent Control: Integration Tests Validate

These rules are ALWAYS ACTIVE for all AI agent configuration systems that support MCP (Model Context Protocol) integration with nested directory structures, including projects with per-agent MCP server configurations distributed across project root, module, and submodule directories.

### Rules

- **R-MCP-001** SHOULD: Integration tests SHOULD validate nested MCP propagation behavior using describe/it test frameworks with beforeAll/afterAll lifecycle hooks.

### Verify

```bash
# Verify nested MCP tests are present and use proper lifecycle management
grep -r 'applyAllAgentConfigs.*nested.*true' tests/integration/ || echo 'No nested MCP tests found'

# Count ruler.toml files with agent MCP configuration sections
find . -name 'ruler.toml' -exec grep -l '\[agents\..*\.mcp\]' {} \; | wc -l

# Validate MCP JSON files have correct structure with mcpServers or servers keys
find . -name '.mcp.json' -o -name '.mcp.json.bak' | while read f; do jq -e '.mcpServers // .servers' "$f" > /dev/null && echo "$f: valid" || echo "$f: invalid"; done
```

**Accept when:**
- Integration tests validate that MCP configuration files are created only in directories where the corresponding agent has mcp.enabled=true in ruler.toml
- Tests confirm that MCP server definitions remain isolated to their defining directory without appearing in parent or child directory configurations
- Backup files with .bak extension are created before overwriting existing MCP configurations, and both original and backup paths are added to .gitignore
- Test fixtures use beforeAll/afterAll lifecycle hooks to manage setup and teardown of nested directory structures
- Agent-specific enablement flags (e.g., agents.copilot.mcp.enabled=false) are honored and prevent MCP file creation in scoped directories

<enforcement>
Claude Code MUST NOT skip or defer verification of nested MCP configuration behavior. Integration test failures blocking PR merges when nested MCP propagation deviates from expected patterns are mandatory. Configuration validation errors must be logged with specific directory and agent context.
</enforcement>