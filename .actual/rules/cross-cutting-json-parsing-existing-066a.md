# Isolate MCP Server Configuration State Using Set-Based Deduplication: Json Parsing Existing

These rules are ALWAYS ACTIVE for all agent configuration management code that handles MCP server definitions and gitignore entry tracking across nested directory hierarchies.

### Rules

- **R-MCP-JSON-001** MUST: JSON parsing of existing MCP configuration files MUST be wrapped in try-catch blocks with explicit error logging before state mutation.
- **R-MCP-JSON-002** MUST: Use Set data structures (Set<string>) for tracking gitignore entries and configuration keys, with explicit .add() calls at each insertion point.
- **R-MCP-JSON-003** MUST: When adding gitignore entries for MCP files, always add both the primary path and the .bak backup path in a single operation.
- **R-MCP-JSON-004** SHOULD: Implement helper functions like extractServers() to normalize access to MCP server definitions across different agent configuration schemas (mcpServers vs servers keys).
- **R-MCP-JSON-005** SHOULD: Validate configuration isolation by asserting that parent directory server keys do not appear in child directory configurations unless explicitly inherited.

### Verify

```bash
# Count Set usage for gitignore/seen/keys tracking
grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|seen|keys)' | wc -l

# Count JSON.parse operations without try-catch in agent code
grep -r 'JSON\.parse' src/agents/ | xargs grep -L 'try.*catch' | wc -l

# Verify .bak backup paths are added with primary paths
grep -r '\.add.*\.bak' tests/integration/ | grep -c 'expectedGitignoreEntries'
```

**Accept when:**
- All MCP configuration aggregation code uses Set data structures for deduplication, verified by grep showing Set usage in relevant files
- All JSON.parse operations on agent configuration files are wrapped in try-catch blocks with error handling
- Gitignore entry addition includes both primary and .bak backup paths, verified in integration tests
- Integration tests validate no duplicate MCP server keys appear in generated configuration files
- Configuration isolation is verified: parent directory server keys do not appear in child directory configurations unless explicitly inherited

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON parsing operations in MCP configuration code paths MUST be wrapped in try-catch blocks before merging. Set-based deduplication MUST be used for gitignore entries and configuration keys. Integration tests MUST pass with zero duplicate entries detected.
</enforcement>