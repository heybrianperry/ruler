# Use Map-Based In-Memory Caching for MCP Server Registry State: Configuration Parsing Filesystem

These rules are ALWAYS ACTIVE for all MCP server registry and configuration management code, including functions handling server lifecycle (propagateMcpToOpenHands, readVSCodeSettings, writeVSCodeSettings, transformRulerToAugmentMcp) and transport-specific server collections for SSE, SHTTP, and STDIO protocols.

### Rules

- **R-CACHE-001** MUST: Configuration parsing from filesystem sources MUST occur before cache population, using appropriate parsers (parseTOML for TOML files, JSON.parse for JSON files).
- **R-CACHE-002** MUST: Use Map instances for in-memory server registry state, keyed by URL string for SSE/SHTTP transports and by name string for STDIO transport.
- **R-CACHE-003** MUST: Initialize Map instances at module scope or in constructor functions to ensure single registry instance per transport type.
- **R-CACHE-004** MUST: Wrap filesystem parsing operations (parseTOML, JSON.parse) in try-catch blocks and validate structure before Map.set() to prevent invalid state.
- **R-CACHE-005** SHOULD: Use descriptive variable names that indicate both the data structure and purpose (existingSseServers, existingShttpServers, existingStdioServers, existingServerMap).
- **R-CACHE-006** SHOULD: Implement helper functions like getOrCreateServer() that encapsulate Map.has() + Map.get() + Map.set() patterns for consistency.
- **R-CACHE-007** SHOULD: Document the key schema for each Map instance in module-level comments.

### Verify

```bash
# Verify Map usage for server collections
grep -r 'Map<.*>' src/mcp/ src/vscode/ --include='*.ts' | grep -E '(Server|server)' | wc -l

# Verify Map.set() usage in identified modules
grep -r '\.set(' src/mcp/propagateOpenHandsMcp.ts src/vscode/settings.ts | grep -E '(existingSseServers|existingShttpServers|existingStdioServers|existingServerMap)' | wc -l

# Verify configuration parsing precedes cache operations
grep -r 'parseTOML\|JSON\.parse' src/mcp/propagateOpenHandsMcp.ts src/vscode/settings.ts | wc -l
```

**Accept when:**
- All MCP server registry modules use Map instances for in-memory state, verified by grep finding Map type declarations for server collections
- Server registration operations use Map.set() method, verified by grep finding at least 5 occurrences across propagateOpenHandsMcp.ts and settings.ts
- Configuration parsing precedes cache operations, verified by presence of parseTOML and JSON.parse calls in the same modules
- Filesystem parsing operations are wrapped in try-catch blocks before Map.set() calls
- Module-level comments document the key schema for each Map instance

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Code review checklist MUST require Map usage for new server registry implementations. Static analysis rules MUST detect plain object usage for server collections. Unit tests MUST verify Map.set() and Map.get() behavior for server lifecycle operations.
</enforcement>