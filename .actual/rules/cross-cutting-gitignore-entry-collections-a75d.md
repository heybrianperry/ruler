# Isolate MCP Server Configuration State Using Set-Based Deduplication: Gitignore Entry Collections

These rules are ALWAYS ACTIVE for all MCP configuration management code paths that handle server registration, gitignore updates, and configuration state tracking across nested directory hierarchies.

### Rules

- **R-GITIGNORE-001** MUST: All gitignore entry collections for MCP configuration files and backup files MUST use Set data structures to prevent duplicate entries.

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
- No duplicate gitignore entries are detected in generated configuration files

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration tests MUST validate gitignore content uniqueness. Code review MUST enforce Set usage for configuration state collections. CI pipeline MUST fail if duplicate gitignore entries are detected.
</enforcement>