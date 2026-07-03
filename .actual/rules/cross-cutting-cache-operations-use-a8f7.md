# Use Map-Based Cache Layer for Server Configuration State: Cache Operations Use

These rules are ALWAYS ACTIVE for all configuration management modules that handle server definitions, including VSCode settings management and OpenHands MCP propagation code.

### Rules

- **R-CACHE-001** MUST: Cache operations MUST use Map.set(key, value) to store or update server entries where key is derived from server.name or server URL properties.

### Verify

```bash
# Check for Map usage in configuration modules
grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations with server-specific keys
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|url|newServer\.name)'

# Count Map instances in codebase
grep -r 'new Map' src/ | wc -l
```

**Accept when:**
- All server configuration modules use Map instances for caching server entries with keys derived from server.name or url properties.
- Map.set() operations are present for all server configuration updates in readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands functions.
- No direct array or object mutations bypass the Map cache layer in configuration synchronization code paths.

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based cache layer usage in configuration management modules. All configuration state mutations MUST flow through Map.set() operations with properly derived keys.
</enforcement>