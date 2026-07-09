# Use Set-Based Deduplication for Message Queue and Path Collection: Convert Sets Arrays

These rules are ALWAYS ACTIVE for all code that collects file paths, gitignore entries, or configuration keys where uniqueness must be guaranteed.

### Rules

- **R-DEDUP-001** MAY: Convert Sets to arrays for final output or iteration when order-independent uniqueness is the primary requirement.
- **R-DEDUP-002** MUST: Initialize Sets at the beginning of functions that traverse nested directories (e.g., `const pathSet = new Set<string>()`).
- **R-DEDUP-003** MUST: Use `pathSet.add(path.join(...))` patterns to add normalized paths during traversal.
- **R-DEDUP-004** MUST: Convert Sets to arrays for final return values using `Array.from(set)` or `[...set]` spread syntax.
- **R-DEDUP-005** MUST: Wrap `JSON.parse()` calls in try-catch blocks when reading existing configuration files, defaulting to empty objects on parse failure.
- **R-DEDUP-006** SHOULD: Use `Set.has()` to check for duplicates before expensive operations like file I/O.

### Verify

```bash
# Verify Set usage for path/key collection
grep -r 'new Set<' src/ tests/ | grep -E '(path|gitignore|seen|targets)' | wc -l

# Verify add() method calls on Sets
grep -r '\.add\(' src/ tests/ | grep -E '(Set|pathSet|seen|targets|expectedGitignoreEntries)' | wc -l

# Verify JSON.parse() patterns with error handling
grep -r 'JSON\.parse' src/ | grep -E '(fs\.readFile|existingContent|content)' | wc -l
```

**Accept when:**
- All three grep commands return non-zero counts, confirming Set usage, add() method calls, and JSON.parse() patterns exist.
- Code review confirms no array-based path collection in nested directory traversal functions.
- Integration tests pass for nested MCP configuration generation without duplicate gitignore entries.
- No duplicate gitignore entries are detected across nested project structures.

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must return non-zero counts before accepting code that implements path or configuration key collection. Integration tests validating gitignore uniqueness must pass. Code review must confirm Set-based deduplication is used instead of array-based collection patterns.
</enforcement>