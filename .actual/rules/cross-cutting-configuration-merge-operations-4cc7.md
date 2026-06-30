# Isolate MCP Server Configuration State Using Set-Based Deduplication: Configuration Merge Operations

These rules are ALWAYS ACTIVE for all agent configuration management code that handles MCP server definitions and gitignore entry tracking across nested directory hierarchies.

### Rules

- **R-MCP-001** SHOULD: Configuration merge operations SHOULD validate that parent directory servers do not leak into child directory configurations unless explicitly inherited.

### Verify

```bash
# Verify Set-based deduplication is used for gitignore and configuration key tracking
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
- Integration tests validate no duplicate MCP server keys appear in generated configuration files
- Parent directory server keys do not appear in child directory configurations unless explicitly inherited

<enforcement>
Claude Code MUST NOT skip or defer verification. All MCP configuration merge operations MUST be validated against these rules before merge. Violations block CI pipeline and require engineering lead approval for exceptions.
</enforcement>