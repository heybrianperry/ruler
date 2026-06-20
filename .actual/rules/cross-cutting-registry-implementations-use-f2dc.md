# Use Map-Based In-Memory Caching for MCP Server Registry State: Registry Implementations Use

These rules are ALWAYS ACTIVE for all MCP server registry and configuration management code, including modules handling server lifecycle events (registration, lookup, updates) across SSE, SHTTP, and STDIO transport types.

### Rules

- **R-REGISTRY-001** MAY: Registry implementations MAY use Map.get() for lookups and Map.has() for existence checks to leverage native Map performance characteristics.

### Verify

```bash
# Verify Map-based server collections exist in registry modules
grep -r 'Map<.*>' src/mcp/ src/vscode/ --include='*.ts' | grep -E '(Server|server)' | wc -l

# Verify Map.set() usage in identified registry files
grep -r '\.set(' src/mcp/propagateOpenHandsMcp.ts src/vscode/settings.ts | grep -E '(existingSseServers|existingShttpServers|existingStdioServers|existingServerMap)' | wc -l

# Verify configuration parsing precedes cache operations
grep -r 'parseTOML\|JSON\.parse' src/mcp/propagateOpenHandsMcp.ts src/vscode/settings.ts | wc -l
```

**Accept when:**
- All MCP server registry modules use Map instances for in-memory state, verified by grep finding Map type declarations for server collections
- Server registration operations use Map.set() method, verified by grep finding at least 5 occurrences across propagateOpenHandsMcp.ts and settings.ts
- Configuration parsing precedes cache operations, verified by presence of parseTOML and JSON.parse calls in the same modules

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based caching patterns in MCP server registry implementations. All three verify commands MUST pass before accepting code changes to registry modules.
</enforcement>