# Isolate MCP Server Configuration State Using Set-Based Deduplication: Agent Specific Mcp

These rules are ALWAYS ACTIVE for all MCP configuration management code paths that handle server registration, gitignore updates, and configuration state tracking across nested directory hierarchies.

### Rules

- **R-MCP-001** SHOULD: Agent-specific MCP configuration paths SHOULD be normalized to relative paths with forward slashes before adding to gitignore Sets.

### Verify

```bash
# Verify Set-based deduplication patterns in gitignore and MCP configuration code
grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|mcp|config)' | wc -l

# Verify all JSON.parse operations are wrapped in try-catch blocks
grep -r 'JSON\.parse' src/ | xargs grep -L 'try\|catch' | wc -l

# Verify Set.add() patterns are present in configuration state tracking
grep -r '\.add(' src/ tests/ | grep -E '(expectedGitignoreEntries|seen)' | wc -l
```

**Accept when:**
- All gitignore entry collections use Set data structures (first verify command returns > 0)
- All JSON.parse operations are wrapped in try-catch blocks (second verify command returns 0)
- Set.add() patterns are present in configuration state tracking code (third verify command returns > 0)
- Integration tests validate gitignore content uniqueness across nested directory configurations
- No duplicate gitignore entries exist across nested directory hierarchies with multiple agent configurations

<enforcement>
Claude Code MUST NOT skip or defer verification. All MCP configuration code paths MUST use Set-based deduplication for gitignore entries and configuration state tracking. CI pipeline MUST fail if integration tests detect duplicate gitignore entries. Code review MUST block merges introducing array-based deduplication patterns in MCP configuration paths.
</enforcement>