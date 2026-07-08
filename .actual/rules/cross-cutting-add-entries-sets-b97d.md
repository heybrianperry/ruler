# Use Set-Based Deduplication for Message Queue and Path Collection: Add Entries Sets

These rules are ALWAYS ACTIVE for all code that collects file paths, gitignore entries, or configuration keys where uniqueness must be guaranteed, including file path collection for gitignore updates, MCP server key aggregation across nested configurations, skills manifest entry tracking, configuration backup path generation, and agent-specific configuration file path resolution.

### Rules

- **R-DEDUP-001** MUST: Add entries to Sets using the add() method rather than array push operations when deduplication is required.

### Verify

```bash
# Verify Set usage patterns exist in codebase
grep -r 'new Set<' src/ tests/ | grep -E '(path|gitignore|seen|targets)' | wc -l

# Verify add() method calls on Sets
grep -r '\.add\(' src/ tests/ | grep -E '(Set|pathSet|seen|targets|expectedGitignoreEntries)' | wc -l

# Verify JSON.parse() patterns with error handling
grep -r 'JSON\.parse' src/ | grep -E '(fs\.readFile|existingContent|content)' | wc -l
```

**Accept when:**
- All three grep commands return non-zero counts, confirming Set usage, add() method calls, and JSON.parse() patterns exist
- Code review confirms no array-based path collection in nested directory traversal functions
- Integration tests pass for nested MCP configuration generation without duplicate gitignore entries
- JSON.parse() calls are wrapped in try-catch blocks with appropriate error handling and fallback to empty objects

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must return non-zero counts and integration tests must pass before accepting code that collects paths, gitignore entries, or configuration keys.
</enforcement>