# Use Map-Based In-Memory Cache for Server Configuration State: Cache Updates Use

These rules are ALWAYS ACTIVE for all files matching server configuration state management, including src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts, and any new code paths that handle server configuration caching and updates.

### Rules

- **R-CACHE-001** MUST: Cache updates MUST use Map.set() operations to ensure idempotent insertion and update semantics.

### Verify

```bash
# Verify Map-based cache patterns are present in server configuration code
grep -r 'new Map()' src/ | grep -E '(Server|server)' | wc -l

# Verify Map.set() operations are used for server cache updates
grep -r '\.set\(' src/ | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)' | wc -l

# Verify no linear Array.find() patterns exist for server lookups
grep -r 'Array\.find\(' src/ | grep -E '(server|Server)' | wc -l
```

**Accept when:**
- Map-based cache patterns are detected in at least 2 files handling server configuration (verified by grep showing Map instantiation and .set() operations)
- No linear Array.find() patterns exist for server lookups in configuration management code paths
- All server configuration updates use Map.set() with consistent key strategies (name for stdio, url for SSE/SHTTP)

<enforcement>
Claude Code MUST NOT skip or defer verification. All server configuration cache updates MUST use Map.set() operations. Violations detected by grep-based verification commands in CI pipeline will cause build failure. Code review requires justification and architectural approval for any alternative caching strategies.
</enforcement>