# Isolate MCP Server Configuration State Using Set-Based Deduplication: Gitignore Entries Mcp

These rules are ALWAYS ACTIVE for all agent configuration management code that handles MCP server definitions and gitignore entry tracking across nested directory hierarchies.

### Rules

- **R-MCP-GITIGNORE-001** MUST: Gitignore entries for MCP configuration files MUST include both the primary file path and backup file path (with .bak extension) in the deduplication set.
- **R-MCP-GITIGNORE-002** MUST: All JSON.parse() calls for MCP configuration files (.mcp.json, firebender.json) MUST be wrapped in try-catch blocks with error handling to prevent state corruption.
- **R-MCP-GITIGNORE-003** MUST: Configuration aggregation code MUST use TypeScript Set<string> data structures for tracking gitignore entries and configuration keys, with explicit .add() calls at each insertion point.
- **R-MCP-GITIGNORE-004** MUST: MCP server definitions MUST be isolated by directory level (projectRoot, moduleDir, submoduleDir) with no leakage of server keys between levels unless explicitly inherited.
- **R-MCP-GITIGNORE-005** SHOULD: Implement validation that compares server definitions before deduplication and logs warnings when conflicting definitions are merged.

### Verify

```bash
# Verify Set-based deduplication is used for gitignore and configuration tracking
grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|seen|keys)' | wc -l

# Verify all JSON.parse operations on agent configuration files are wrapped in try-catch
grep -r 'JSON\.parse' src/agents/ | xargs grep -L 'try.*catch' | wc -l

# Verify gitignore entry addition includes both primary and .bak backup paths
grep -r '\.add.*\.bak' tests/integration/ | grep -c 'expectedGitignoreEntries'
```

**Accept when:**
- All MCP configuration aggregation code uses Set data structures for deduplication, verified by grep showing Set usage in relevant files
- All JSON.parse operations on agent configuration files are wrapped in try-catch blocks with error handling
- Gitignore entry addition includes both primary and .bak backup paths, verified in integration tests
- Integration tests validate that no duplicate MCP server keys appear in generated configuration files
- MCP server definitions do not leak between directory levels (rootServers do not contain module-specific keys, moduleServers do not contain root-specific keys)

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code paths that write MCP configuration files or aggregate gitignore entries. Violations must be caught during code review and CI pipeline execution.
</enforcement>