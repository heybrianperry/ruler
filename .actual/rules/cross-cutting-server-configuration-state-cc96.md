# Use Map-Based Cache Layer for Server Configuration State: Server Configuration State

These rules are ALWAYS ACTIVE for all server configuration management code in `src/vscode/settings.ts`, `src/mcp/propagateOpenHandsMcp.ts`, and any new configuration modules that read, modify, or write server definitions (VSCode MCP servers, OpenHands SSE/SHTTP/STDIO servers).

### Rules

- **R-CONFIG-001** MUST: Server configuration state MUST be cached in Map instances keyed by unique server identifiers (name for STDIO servers, url for SSE/SHTTP servers).
- **R-CONFIG-002** MUST: Initialize Map instances with descriptive names (existingServerMap, existingSseServers, existingStdioServers) before reading configuration files to establish clear cache boundaries.
- **R-CONFIG-003** MUST: Use consistent key derivation: server.name for STDIO servers, serverDef.url or url property for SSE/SHTTP servers.
- **R-CONFIG-004** MUST: Populate Map caches immediately after parsing configuration files (JSON.parse for VSCode, parseTOML for OpenHands) to capture existing state before modifications.
- **R-CONFIG-005** MUST: Convert Map entries back to configuration format using Array.from(map.values()) or map iteration before serializing to JSON/TOML for disk writes.
- **R-CONFIG-006** SHOULD: Implement transactional write patterns with atomic file operations (write to temp file, then rename) and error handling that invalidates cache on write failure.
- **R-CONFIG-007** SHOULD: Implement cache lifecycle management with explicit clear() calls before repopulating from disk, or use scoped Map instances within function boundaries.
- **R-CONFIG-008** SHOULD: Add key uniqueness assertions during Map.set() operations to prevent server configurations from overwriting each other.

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
- Map caches are populated immediately after parsing configuration files and cleared or scoped appropriately in long-running processes.
- Configuration write operations use atomic file patterns with error handling that invalidates cache on failure.

<enforcement>
Clause Code MUST NOT skip or defer verification of Map-based caching patterns in server configuration modules. Violations require code review feedback requesting refactoring to Map-based cache pattern, CI pipeline warnings, or architecture review escalation for persistent deviations.
</enforcement>