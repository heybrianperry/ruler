# Adopt Nested MCP Configuration with Per-Directory Agent Control: Mcp Configuration Files

These rules are ALWAYS ACTIVE for all AI agent configuration systems that support MCP (Model Context Protocol) integration with nested directory structures, including projects with per-agent MCP server configurations distributed across project root, module, and submodule directories.

### Rules

- **R-MCP-001** MUST: MCP configuration files MUST be valid JSON with either `mcpServers` or `servers` top-level keys containing server definitions.
- **R-MCP-002** MUST: MCP configuration files MUST be created only in directories where the corresponding agent has `mcp.enabled=true` in the local `ruler.toml`.
- **R-MCP-003** MUST: MCP server definitions MUST remain isolated to their defining directory without appearing in parent or child directory configurations.
- **R-MCP-004** MUST: Backup files with `.bak` extension MUST be created before overwriting existing MCP configurations.
- **R-MCP-005** MUST: Both original MCP config paths and their `.bak` backup paths MUST be added to `.gitignore` using relative paths from project root.
- **R-MCP-006** SHOULD: Use the `applyAllAgentConfigs` function with `nested=true` flag to enable per-directory MCP configuration mode.
- **R-MCP-007** SHOULD: Implement `getNativeMcpPath` helper to resolve agent-specific MCP configuration file locations across different directory contexts.
- **R-MCP-008** SHOULD: Parse MCP JSON files using `extractServers` helper that handles both `mcpServers` and `servers` key variations for cross-agent compatibility.

### Verify

```bash
# Verify nested MCP configuration tests are present
grep -r 'applyAllAgentConfigs.*nested.*true' tests/integration/ || echo 'No nested MCP tests found'

# Count ruler.toml files with MCP agent configuration sections
find . -name 'ruler.toml' -exec grep -l '\[agents\..*\.mcp\]' {} \; | wc -l

# Validate all MCP JSON files have correct structure and contain mcpServers or servers key
find . -name '.mcp.json' -o -name '.mcp.json.bak' | while read f; do jq -e '.mcpServers // .servers' "$f" > /dev/null && echo "$f: valid" || echo "$f: invalid"; done
```

**Accept when:**
- Integration tests validate that MCP configuration files are created only in directories where the corresponding agent has `mcp.enabled=true` in `ruler.toml`
- Tests confirm that MCP server definitions remain isolated to their defining directory without appearing in parent or child directory configurations
- Backup files with `.bak` extension are created before overwriting existing MCP configurations
- Both original and backup MCP config paths are added to `.gitignore` using relative paths from project root
- All MCP JSON files pass validation for either `mcpServers` or `servers` top-level key presence
- Per-directory agent enablement flags are honored (e.g., `agents.copilot.mcp.enabled=false` in module directory prevents Copilot MCP file creation there)

<enforcement>
Claude Code MUST NOT skip or defer verification of nested MCP configuration rules. Integration test failures blocking PR merges when nested MCP propagation behavior deviates from expected patterns are mandatory. Configuration validation errors MUST be logged with specific directory and agent context to aid debugging.
</enforcement>