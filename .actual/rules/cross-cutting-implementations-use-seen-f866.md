# Isolate MCP Server Configuration State Using Set-Based Deduplication: Implementations Use Seen

These rules are ALWAYS ACTIVE for all MCP configuration management code paths that handle server registration, gitignore updates, and configuration state tracking across nested directory hierarchies.

### Rules

- **R-MCP-SET-001** MAY: Implementations MAY use seen.add(key) patterns for tracking processed configuration keys in concurrent or iterative processing contexts.
- **R-MCP-SET-002** MUST: Initialize Set collections at the beginning of configuration processing functions using const seen = new Set<string>().
- **R-MCP-SET-003** MUST: Use Set.add() for all gitignore entry accumulation and configuration key tracking operations.
- **R-MCP-SET-004** MUST: Convert Sets to arrays using Array.from(set) or [...set] before JSON serialization or file writing.
- **R-MCP-SET-005** MUST: Wrap all JSON.parse calls in try-catch blocks with console.warn or logger.error for parse failures.
- **R-MCP-SET-006** MUST: Normalize file paths to relative forward-slash format before adding to gitignore Sets to ensure cross-platform consistency.
- **R-MCP-SET-007** SHOULD: Convert Set to sorted array before writing to files to ensure deterministic gitignore file ordering.

### Verify

```bash
# Count Set usage in gitignore/mcp/config contexts
grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|mcp|config)' | wc -l

# Count JSON.parse operations without try-catch protection
grep -r 'JSON\.parse' src/ | xargs grep -L 'try\|catch' | wc -l

# Count Set.add() patterns in configuration state tracking
grep -r '\.add(' src/ tests/ | grep -E '(expectedGitignoreEntries|seen)' | wc -l
```

**Accept when:**
- All gitignore entry collections use Set data structures (verify command 1 returns > 0)
- All JSON.parse operations are wrapped in try-catch blocks (verify command 2 returns 0)
- Set.add() patterns are present in configuration state tracking code (verify command 3 returns > 0)
- Integration tests validate gitignore content uniqueness across nested directory configurations
- No duplicate gitignore entries exist across nested directory hierarchies with multiple agent configurations

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration tests MUST validate gitignore content uniqueness. CI pipeline MUST fail if duplicate gitignore entries are detected. Code review MUST block merges introducing array-based deduplication patterns in MCP configuration paths.
</enforcement>