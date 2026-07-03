# Use Map-Based Cache Layer for Server Configuration State: Configuration Read Operations

These rules are ALWAYS ACTIVE for all configuration management modules that read, parse, and synchronize server definitions from disk-based configuration files (JSON for VSCode settings, TOML for OpenHands) into runtime state.

### Rules

- **R-CONFIG-001** MUST: Configuration read operations MUST populate the Map cache before performing any modifications to preserve existing entries.

### Verify

```bash
# Verify Map-based caching is used in configuration modules
grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations are present for server configuration updates
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|url|newServer\.name)'

# Count Map instances in codebase
grep -r 'new Map' src/ | wc -l
```

**Accept when:**
- All server configuration modules use Map instances for caching server entries with keys derived from server.name or url properties.
- Map.set() operations are present for all server configuration updates in readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands functions.
- No direct array or object mutations bypass the Map cache layer in configuration synchronization code paths.
- Map caches are populated immediately after parsing configuration files and before any modifications occur.
- Existing server entries are preserved during partial configuration updates through Map-based state management.

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based cache layer usage in configuration read operations. All configuration management code must be reviewed to ensure compliance with R-CONFIG-001 before merge.
</enforcement>