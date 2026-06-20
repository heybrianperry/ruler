# Adopt Nested MCP Configuration with Per-Directory Agent Control: Test Fixtures Create

These rules are ALWAYS ACTIVE for all AI agent configuration systems that support MCP (Model Context Protocol) integration with nested directory structures, including test fixture creation and validation across multi-level directory hierarchies.

### Rules

- **R-MCP-NEST-001** SHOULD: Test fixtures SHOULD create multi-level directory structures (root/module/submodule) with varying agent enablement patterns to verify isolation of MCP configurations across nested contexts.

### Verify

```bash
# Verify nested MCP test fixtures are present
grep -r 'applyAllAgentConfigs.*nested.*true' tests/integration/ || echo 'No nested MCP tests found'

# Count ruler.toml files with MCP agent sections
find . -name 'ruler.toml' -exec grep -l '\[agents\..*\.mcp\]' {} \; | wc -l

# Validate MCP JSON file structure and server definitions
find . -name '.mcp.json' -o -name '.mcp.json.bak' | while read f; do jq -e '.mcpServers // .servers' "$f" > /dev/null && echo "$f: valid" || echo "$f: invalid"; done
```

**Accept when:**
- Integration tests validate that MCP configuration files are created only in directories where the corresponding agent has mcp.enabled=true in ruler.toml
- Tests confirm that MCP server definitions remain isolated to their defining directory without appearing in parent or child directory configurations
- Backup files with .bak extension are created before overwriting existing MCP configurations, and both original and backup paths are added to .gitignore
- Multi-level directory structures (projectRoot, moduleDir, submoduleDir) are created with independent ruler.toml files demonstrating per-directory configuration isolation
- Agent-specific enablement flags (e.g., agents.copilot.mcp.enabled=false in module directory) prevent MCP file creation in that context while allowing it in other contexts

<enforcement>
Claude Code MUST NOT skip or defer verification of nested MCP configuration test fixtures. Integration test failures blocking PR merges when nested MCP propagation behavior deviates from expected patterns are mandatory enforcement points.
</enforcement>