# Use Map-Based In-Memory Caching for MCP Server Registry State: Mcp Server Registry

These rules are ALWAYS ACTIVE for all MCP server registry and configuration management code, including modules handling server lifecycle events, transport-specific server collections, and configuration parsing operations.

### Rules

- **R-MCPREG-001** MUST: MCP server registry implementations MUST use JavaScript Map instances for in-memory caching of server state, keyed by server identifier (URL for SSE/SHTTP transports, name for STDIO transport).
- **R-MCPREG-002** MUST: Initialize Map instances at module scope or in constructor functions to ensure single registry instance per transport type.
- **R-MCPREG-003** MUST: Wrap filesystem parsing operations (parseTOML, JSON.parse) in try-catch blocks and validate structure before Map.set() to prevent invalid state.
- **R-MCPREG-004** SHOULD: Use descriptive variable names that indicate both the data structure and purpose: existingSseServers, existingShttpServers, existingStdioServers, existingServerMap.
- **R-MCPREG-005** SHOULD: Implement helper functions like getOrCreateServer() that encapsulate Map.has() + Map.get() + Map.set() patterns for consistency.
- **R-MCPREG-006** SHOULD: Document the key schema for each Map instance (URL string for SSE/SHTTP, name string for STDIO) in module-level comments.
- **R-MCPREG-007** SHOULD: Implement explicit cleanup methods that call Map.delete() when servers are removed to prevent memory leaks.
- **R-MCPREG-008** SHOULD: Document Map access patterns as single-threaded; if concurrency is required, wrap Map operations in mutex or use atomic update patterns.

### Verify

```bash
# Verify Map type declarations for server collections
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
- No plain JavaScript objects are used for server registry state without approved exception EXC-001

<enforcement>
Clause Code MUST NOT skip or defer verification. All new MCP server registry implementations must pass the verify commands before merge. Static analysis rules detecting plain object usage for server collections must be treated as blocking errors in CI.
</enforcement>