# Isolate MCP Server Configuration State Using Set-Based Deduplication: Configuration Application Functions

These rules are ALWAYS ACTIVE for all agent configuration management code that handles MCP server definitions and gitignore entry tracking across multiple agents (Claude, Copilot, Windsurf) and nested directory hierarchies.

### Rules

- **R-MCP-001** MUST NOT: Configuration application functions MUST NOT allow duplicate MCP server keys within a single agent's configuration scope.
- **R-MCP-002** MUST: Use TypeScript Set<string> for tracking gitignore entries and configuration keys, with explicit .add() calls at each insertion point.
- **R-MCP-003** MUST: Wrap all JSON.parse() calls for MCP configuration files in try-catch blocks with console.warn or structured logging on failure.
- **R-MCP-004** MUST: When adding gitignore entries for MCP files, always add both the primary path and the .bak backup path in a single operation.
- **R-MCP-005** SHOULD: Implement helper functions like extractServers() to normalize access to MCP server definitions across different agent configuration schemas (mcpServers vs servers keys).
- **R-MCP-006** SHOULD: Validate configuration isolation by asserting that parent directory server keys do not appear in child directory configurations unless explicitly inherited.
- **R-MCP-007** SHOULD: Implement validation that compares server definitions before deduplication and logs warnings when conflicting definitions are merged.
- **R-MCP-008** SHOULD: Implement graceful degradation where JSON parsing errors are logged but allow new configuration to be written, with backup files preserved for recovery.

### Verify

```bash
# Count Set usage for gitignore and configuration tracking
grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|seen|keys)' | wc -l

# Count JSON.parse operations without try-catch in agent configuration code
grep -r 'JSON\.parse' src/agents/ | xargs grep -L 'try.*catch' | wc -l

# Verify gitignore entry addition includes both primary and .bak backup paths
grep -r '\.add.*\.bak' tests/integration/ | grep -c 'expectedGitignoreEntries'
```

**Accept when:**
- All MCP configuration aggregation code uses Set data structures for deduplication, verified by grep showing Set usage in relevant files
- All JSON.parse operations on agent configuration files are wrapped in try-catch blocks with error handling
- Gitignore entry addition includes both primary and .bak backup paths, verified in integration tests
- Integration tests validate no duplicate MCP server keys appear in generated configuration files
- Configuration isolation is verified by tests asserting that parent directory server keys do not appear in child directory configurations unless explicitly inherited

<enforcement>
Claude Code MUST NOT skip or defer verification. All MCP configuration application code MUST be reviewed against these rules before merge. CI pipeline MUST fail if integration tests detect duplicate MCP server keys in generated configurations. Code review MUST block merge if JSON parsing lacks error handling in agent configuration code.
</enforcement>