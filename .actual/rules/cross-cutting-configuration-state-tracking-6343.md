# Isolate MCP Server Configuration State Using Set-Based Deduplication: Configuration State Tracking

These rules are ALWAYS ACTIVE for all MCP configuration management code paths that handle server registration, gitignore updates, and configuration state tracking across nested directory hierarchies.

### Rules

- **R-MCP-CONFIG-001** SHOULD: Configuration state tracking across nested directories SHOULD use Set.add() operations to accumulate unique entries rather than array concatenation.

### Verify

```bash
# Verify Set-based deduplication patterns in gitignore and configuration tracking
grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|mcp|config)' | wc -l

# Verify all JSON.parse operations are wrapped in try-catch blocks
grep -r 'JSON\.parse' src/ | xargs grep -L 'try\|catch' | wc -l

# Verify Set.add() patterns are present in configuration state tracking code
grep -r '\.add(' src/ tests/ | grep -E '(expectedGitignoreEntries|seen)' | wc -l
```

**Accept when:**
- All gitignore entry collections use Set data structures (first verify command returns > 0)
- All JSON.parse operations are wrapped in try-catch blocks (second verify command returns 0)
- Set.add() patterns are present in configuration state tracking code (third verify command returns > 0)
- Integration tests validate gitignore content uniqueness across nested directory configurations
- No duplicate gitignore entries exist across nested directory hierarchies with multiple agent configurations

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration state tracking code MUST use Set-based deduplication patterns. Code review MUST block merges introducing array-based deduplication in MCP configuration paths. CI pipeline MUST fail if integration tests detect duplicate gitignore entries.
</enforcement>