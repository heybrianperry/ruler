# Use Map-Based In-Memory Caching for MCP Server Registry State: Server Registry Implementations

These rules are ALWAYS ACTIVE for all MCP server registry and configuration management code, including functions handling server lifecycle (propagateMcpToOpenHands, readVSCodeSettings, writeVSCodeSettings, transformRulerToAugmentMcp), transport-specific server collections (SSE, SHTTP, STDIO), and configuration parsing operations.

### Rules

- **R-REGISTRY-001** SHOULD: Server registry implementations SHOULD validate input from filesystem sources before cache insertion to prevent invalid state propagation.

### Verify

```bash
# Verify Map-based server registry usage across MCP modules
grep -r 'Map<.*>' src/mcp/ src/vscode/ --include='*.ts' | grep -E '(Server|server)' | wc -l

# Verify Map.set() usage in identified registry files
grep -r '\.set(' src/mcp/propagateOpenHandsMcp.ts src/vscode/settings.ts | grep -E '(existingSseServers|existingShttpServers|existingStdioServers|existingServerMap)' | wc -l

# Verify configuration parsing precedes cache operations
grep -r 'parseTOML\|JSON\.parse' src/mcp/propagateOpenHandsMcp.ts src/vscode/settings.ts | wc -l
```

**Accept when:**
- All MCP server registry modules use Map instances for in-memory state, verified by grep finding Map type declarations for server collections
- Server registration operations use Map.set() method, verified by grep finding at least 5 occurrences across propagateOpenHandsMcp.ts and settings.ts
- Configuration parsing precedes cache operations, verified by presence of parseTOML and JSON.parse calls in the same modules

<enforcement>
Clause Code MUST NOT skip or defer verification of Map-based caching patterns in server registry implementations. All new server registry code MUST use Map instances for state management unless an approved exception (EXC-001) is documented.
</enforcement>