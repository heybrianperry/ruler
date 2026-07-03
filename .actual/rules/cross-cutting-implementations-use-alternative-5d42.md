# Isolate MCP Server Configuration State Using Set-Based Deduplication: Implementations Use Alternative

These rules are ALWAYS ACTIVE for all agent configuration management code that handles MCP server definitions and gitignore entry tracking across nested directory hierarchies (projectRoot, moduleDir, submoduleDir).

### Rules

- **R-MCP-001** MAY: Implementations MAY use alternative deduplication strategies (Map, object keys) if they provide equivalent uniqueness guarantees to Set-based deduplication for MCP server configurations and gitignore entries.
- **R-MCP-002** MUST: All JSON.parse() calls for MCP configuration files (.mcp.json, firebender.json) must be wrapped in try-catch blocks with console.warn or structured logging on failure to prevent silent state corruption.
- **R-MCP-003** MUST: When adding gitignore entries for MCP configuration files, always add both the primary path and the .bak backup path in a single deduplication operation.
- **R-MCP-004** MUST: Validate configuration isolation by asserting that parent directory server keys do not appear in child directory configurations unless explicitly inherited.
- **R-MCP-005** SHOULD: Implement helper functions like extractServers() to normalize access to MCP server definitions across different agent configuration schemas (mcpServers vs servers keys).
- **R-MCP-006** SHOULD: Implement validation that compares server definitions before deduplication and logs warnings when conflicting definitions are merged.

### Verify

```bash
# Count Set-based deduplication usage for gitignore and configuration keys
grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|seen|keys)' | wc -l

# Count JSON.parse operations without try-catch in agent configuration code
grep -r 'JSON\.parse' src/agents/ | xargs grep -L 'try.*catch' | wc -l

# Verify gitignore entry addition includes both primary and .bak backup paths
grep -r '\.add.*\.bak' tests/integration/ | grep -c 'expectedGitignoreEntries'

# Verify no duplicate MCP server keys in generated configuration files
grep -r 'mcpServers\|servers' .actual/generated/ | sort | uniq -d | wc -l
```

**Accept when:**
- All MCP configuration aggregation code uses Set data structures (or equivalent Map/object-key alternatives) for deduplication, verified by grep showing usage in relevant files
- All JSON.parse operations on agent configuration files are wrapped in try-catch blocks with error handling
- Gitignore entry addition includes both primary and .bak backup paths, verified in integration tests
- Integration tests validate no duplicate MCP server keys appear in generated configuration files
- Configuration isolation is verified: parent directory server keys do not leak into child directory configurations

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for MCP configuration management code. Violations must be caught during code review and CI pipeline execution.
</enforcement>