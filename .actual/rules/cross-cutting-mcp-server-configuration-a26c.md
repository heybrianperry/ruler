# Use Map-Based In-Memory Cache for MCP Server Configuration State: Mcp Server Configuration

These rules are ALWAYS ACTIVE for all MCP server configuration read/write operations in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts, including public API functions: readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands, and transformRulerToAugmentMcp.

### Rules

- **R-MCP-CONFIG-001** MUST: MCP server configuration state MUST be cached in-memory using Map data structures keyed by server name for STDIO transport or URL for SSE/SHTTP transports.

### Verify

```bash
# Verify Map usage in configuration modules
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations with appropriate keys
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|newServer\.name|url|serverDef\.url)'

# Verify Map usage in public API functions
grep -r 'readVSCodeSettings\|writeVSCodeSettings\|propagateMcpToOpenHands' src/ --include='*.ts' -A 20 | grep 'Map'
```

**Accept when:**
- All MCP server configuration read/write operations in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts use Map-based caching with appropriate key strategies (name for STDIO, URL for SSE/SHTTP).
- Map.set() operations are present for upserting server entries with correct key selection based on transport type.
- No persistent or application-wide cache instances are introduced for MCP server configuration state.
- Map instances are function-scoped and initialized immediately after parsing configuration files.
- Conversion from Map to array or object structure using Array.from(map.values()) or Object.fromEntries() occurs before serialization to JSON/TOML.

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based caching patterns in configuration management code. All grep-based checks MUST pass before accepting configuration changes.
</enforcement>