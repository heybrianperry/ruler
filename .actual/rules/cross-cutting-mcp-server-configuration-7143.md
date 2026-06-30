# Isolate MCP Server Configuration State Using Set-Based Deduplication: Mcp Server Configuration

These rules are ALWAYS ACTIVE for all agent configuration management code that handles MCP server definitions and gitignore entry tracking across multiple agents (Claude, Copilot, Windsurf) with per-directory configuration files.

### Rules

- **R-MCP-001** MUST: All MCP server configuration keys and gitignore file paths MUST be stored in Set data structures to enforce uniqueness during configuration aggregation.
- **R-MCP-002** MUST: All JSON.parse() calls for MCP configuration files (.mcp.json, firebender.json) MUST be wrapped in try-catch blocks with console.warn or structured logging on failure.
- **R-MCP-003** MUST: When adding gitignore entries for MCP files, always add both the primary path and the .bak backup path in a single operation using Set.add().
- **R-MCP-004** MUST: Configuration merge logic that combines MCP server definitions from multiple directory levels (projectRoot, moduleDir, submoduleDir) MUST use Set-based deduplication to prevent duplicate entries.
- **R-MCP-005** SHOULD: Implement helper functions like extractServers() to normalize access to MCP server definitions across different agent configuration schemas (mcpServers vs servers keys).
- **R-MCP-006** SHOULD: Validate configuration isolation by asserting that parent directory server keys do not appear in child directory configurations unless explicitly inherited.

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
- Integration tests validate that no duplicate MCP server keys appear in generated configuration files
- Configuration isolation is verified by tests asserting that parent directory server keys do not appear in child directory configurations unless explicitly inherited

<enforcement>
Claude Code MUST NOT skip or defer verification. All MCP configuration code MUST comply with R-MCP-001 through R-MCP-006. Violations block CI pipeline and code review merge. Exceptions require engineering lead approval with documented compensating validation logic.
</enforcement>