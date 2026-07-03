# Use Map-Based In-Memory Caching for MCP Server Registry State: Server Registry Cache

These rules are ALWAYS ACTIVE for all MCP server registry and configuration management code, including functions handling server lifecycle (propagateMcpToOpenHands, readVSCodeSettings, writeVSCodeSettings, transformRulerToAugmentMcp) and transport-specific server collections for SSE, SHTTP, and STDIO protocols.

### Rules

- **R-CACHE-001** MUST: Server registry cache operations MUST use Map.set() for registration and updates to ensure atomic key-value association.

### Verify

```bash
# Verify Map type declarations exist for server collections
grep -r 'Map<.*>' src/mcp/ src/vscode/ --include='*.ts' | grep -E '(Server|server)' | wc -l

# Verify Map.set() usage in identified modules
grep -r '\.set(' src/mcp/propagateOpenHandsMcp.ts src/vscode/settings.ts | grep -E '(existingSseServers|existingShttpServers|existingStdioServers|existingServerMap)' | wc -l

# Verify configuration parsing precedes cache operations
grep -r 'parseTOML\|JSON\.parse' src/mcp/propagateOpenHandsMcp.ts src/vscode/settings.ts | wc -l
```

**Accept when:**
- All MCP server registry modules use Map instances for in-memory state, verified by grep finding Map type declarations for server collections
- Server registration operations use Map.set() method, verified by grep finding at least 5 occurrences across the two identified files
- Configuration parsing precedes cache operations, verified by presence of parseTOML and JSON.parse calls in the same modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to MCP server registry and configuration management code.
</enforcement>