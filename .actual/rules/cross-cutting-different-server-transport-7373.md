# Use Map-Based Cache Layer for Server Configuration State: Different Server Transport

These rules are ALWAYS ACTIVE for all configuration management modules that handle server definitions (VSCode settings, OpenHands MCP servers) requiring efficient lookup and synchronization between disk-based configuration files and runtime state.

### Rules

- **R-CACHE-001** SHOULD: Different server transport types (SSE, SHTTP, STDIO) SHOULD maintain separate Map instances to enforce type safety and prevent key collisions.
- **R-CACHE-002** MUST: Use Map-based caching for server configuration state in all configuration synchronization code paths (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands).
- **R-CACHE-003** MUST: Derive Map keys consistently: server.name for STDIO servers, serverDef.url or url property for SSE/SHTTP servers.
- **R-CACHE-004** MUST: Populate Map caches immediately after parsing configuration files to capture existing state before modifications.
- **R-CACHE-005** MUST: Convert Map entries back to configuration format using Array.from(map.values()) or map iteration before serializing to JSON/TOML for disk writes.
- **R-CACHE-006** SHOULD: Initialize Map instances with descriptive names (existingServerMap, existingSseServers, existingStdioServers) to establish clear cache boundaries.
- **R-CACHE-007** MUST: Implement cache lifecycle management with explicit clear() calls before repopulating from disk in long-running processes.
- **R-CACHE-008** MUST: Implement transactional write patterns with atomic file operations (write to temp file, then rename) and error handling that invalidates cache on write failure.

### Verify

```bash
# Verify Map usage in configuration modules
grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations with proper key derivation
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|url|newServer\.name)'

# Count Map instances in codebase
grep -r 'new Map' src/ | wc -l
```

**Accept when:**
- All server configuration modules use Map instances for caching server entries with keys derived from server.name or url properties.
- Map.set() operations are present for all server configuration updates in readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands functions.
- No direct array or object mutations bypass the Map cache layer in configuration synchronization code paths.
- Separate Map instances exist for different server transport types (SSE, SHTTP, STDIO).
- Map caches are cleared and repopulated on configuration reload cycles.
- Write operations use atomic file patterns with error handling and cache invalidation on failure.

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based caching patterns in configuration management modules. All rules marked MUST are mandatory; SHOULD rules represent strong architectural preferences that require documented justification to override.
</enforcement>