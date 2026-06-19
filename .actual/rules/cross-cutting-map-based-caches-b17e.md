# Use Map-Based Cache Layer for Server Configuration State: Map Based Caches

These rules are ALWAYS ACTIVE for all configuration management modules that handle server definitions, including VSCode settings management and OpenHands MCP propagation code paths.

### Rules

- **R-MAP-001** SHOULD: Map-based caches SHOULD be named to reflect their content scope (e.g., existingServerMap, existingSseServers, existingStdioServers).
- **R-MAP-002** MUST: Use Map instances for caching server entries with keys derived from server.name or url properties in all configuration synchronization code paths.
- **R-MAP-003** MUST: Populate Map caches immediately after parsing configuration files (JSON.parse for VSCode, parseTOML for OpenHands) to capture existing state before modifications.
- **R-MAP-004** MUST: Convert Map entries back to configuration format using Array.from(map.values()) or map iteration before serializing to JSON/TOML for disk writes.
- **R-MAP-005** SHOULD: Use consistent key derivation: server.name for STDIO servers, serverDef.url or url property for SSE/SHTTP servers.
- **R-MAP-006** MUST: Implement transactional write patterns with atomic file operations (write to temp file, then rename) and error handling that invalidates cache on write failure.
- **R-MAP-007** MUST: Enforce strict key derivation rules with validation tests and consider adding key uniqueness assertions during Map.set() operations.
- **R-MAP-008** MUST: Implement cache lifecycle management with explicit clear() calls before repopulating from disk, or use scoped Map instances within function boundaries.

### Verify

```bash
# Verify Map usage in configuration modules
grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations for server configuration updates
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|url|newServer\.name)'

# Count total Map instances in codebase
grep -r 'new Map' src/ | wc -l
```

**Accept when:**
- All server configuration modules use Map instances for caching server entries with keys derived from server.name or url properties.
- Map.set() operations are present for all server configuration updates in readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands functions.
- No direct array or object mutations bypass the Map cache layer in configuration synchronization code paths.
- Transactional write patterns with atomic file operations are implemented for all configuration persistence operations.
- Cache lifecycle management with explicit clear() calls or scoped Map instances is present in long-running processes.

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based cache layer compliance in configuration management modules. All rules marked MUST are non-negotiable; SHOULD rules represent strong preferences subject to documented exceptions.
</enforcement>