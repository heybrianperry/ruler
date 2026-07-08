# Use Set-Based Deduplication for Message Queue and Path Collection: Parse Json Configuration

These rules are ALWAYS ACTIVE for all code that collects file paths, gitignore entries, or configuration keys where uniqueness must be guaranteed, including file path collection for gitignore updates, MCP server key aggregation across nested configurations, skills manifest entry tracking, configuration backup path generation, and agent-specific configuration file path resolution.

### Rules

- **R-DEDUP-001** MUST: Parse JSON configuration files using `JSON.parse()` before merging or extracting server configurations.
- **R-DEDUP-002** MUST: Use `Set` for deduplication of file paths, gitignore entries, and configuration keys during collection in nested directory traversal.
- **R-DEDUP-003** MUST: Initialize Sets at the beginning of functions that traverse nested directories (e.g., `const pathSet = new Set<string>()`).
- **R-DEDUP-004** MUST: Wrap `JSON.parse()` calls in try-catch blocks when reading existing configuration files, defaulting to empty objects on parse failure.
- **R-DEDUP-005** SHOULD: Use `Set.has()` to check for duplicates before expensive operations like file I/O.
- **R-DEDUP-006** SHOULD: Convert Sets to arrays for final return values using `Array.from(set)` or `[...set]` spread syntax.

### Verify

```bash
# Verify Set usage for path/key collection
grep -r 'new Set<' src/ tests/ | grep -E '(path|gitignore|seen|targets)' | wc -l

# Verify add() method calls on Sets
grep -r '\.add\(' src/ tests/ | grep -E '(Set|pathSet|seen|targets|expectedGitignoreEntries)' | wc -l

# Verify JSON.parse() patterns in configuration loading
grep -r 'JSON\.parse' src/ | grep -E '(fs\.readFile|existingContent|content)' | wc -l
```

**Accept when:**
- All three grep commands return non-zero counts, confirming Set usage, add() method calls, and JSON.parse() patterns exist
- Code review confirms no array-based path collection in nested directory traversal functions
- Integration tests pass for nested MCP configuration generation without duplicate gitignore entries
- JSON.parse() failures are wrapped in try-catch blocks with appropriate error handling and logging

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must return non-zero counts and integration tests must pass before accepting code that implements path or configuration key collection.
</enforcement>