# Use Map-Based Cache Layer for Server Configuration State: Configuration Write Operations

These rules are ALWAYS ACTIVE for all configuration management modules that handle server definitions, including VSCode settings management and OpenHands MCP propagation code paths.

### Rules

- **R-CONFIG-001** MUST NOT: Configuration write operations MUST NOT bypass the Map cache; all updates must flow through Map.set() to ensure consistency.

### Verify

```bash
# Verify Map usage in configuration modules
grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations for server configuration updates
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|url|newServer\.name)'

# Count Map instances in codebase
grep -r 'new Map' src/ | wc -l
```

**Accept when:**
- All server configuration modules use Map instances for caching server entries with keys derived from server.name or url properties.
- Map.set() operations are present for all server configuration updates in readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands functions.
- No direct array or object mutations bypass the Map cache layer in configuration synchronization code paths.

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based cache layer compliance in configuration write operations. All configuration state mutations must be validated against R-CONFIG-001 before approval.
</enforcement>