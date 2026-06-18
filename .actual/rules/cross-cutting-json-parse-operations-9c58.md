# Isolate MCP Server Configuration State Using Set-Based Deduplication: Json Parse Operations

These rules are ALWAYS ACTIVE for all MCP configuration management code paths that handle server registration, gitignore updates, and configuration state tracking across nested directory hierarchies.

### Rules

- **R-MCP-JSON-001** MUST: JSON.parse operations on MCP configuration files MUST be wrapped in error handling that logs parse failures and provides fallback behavior.

### Verify

```bash
# Verify Set-based deduplication patterns in gitignore and configuration tracking
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
Claude Code MUST NOT skip or defer verification. All JSON.parse operations in MCP configuration code paths MUST include try-catch error handling with logging. All gitignore and configuration state collections MUST use Set data structures for deduplication. Violations block CI pipeline and code review merge.
</enforcement>