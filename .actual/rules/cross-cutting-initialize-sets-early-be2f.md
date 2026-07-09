# Use Set-Based Deduplication for Message Queue and Path Collection: Initialize Sets Early

These rules are ALWAYS ACTIVE for all code that collects file paths, gitignore entries, or configuration keys where uniqueness must be guaranteed, including nested MCP configuration processing, skills manifest tracking, and multi-agent configuration file generation.

### Rules

- **R-DEDUP-001** SHOULD: Initialize Sets early in collection workflows (e.g., `expectedGitignoreEntries`, `pathSet`, `targets`, `seen`) before iterating over nested structures.
- **R-DEDUP-002** SHOULD: Use `pathSet.add(path.join(...))` patterns to add normalized paths during traversal of nested directories.
- **R-DEDUP-003** SHOULD: Convert Sets to arrays for final return values using `Array.from(set)` or `[...set]` spread syntax.
- **R-DEDUP-004** SHOULD: Wrap `JSON.parse()` calls in try-catch blocks when reading existing configuration files, defaulting to empty objects on parse failure.
- **R-DEDUP-005** SHOULD: Use `Set.has()` to check for duplicates before expensive operations like file I/O.
- **R-DEDUP-006** MUST NOT: Use array-based collection with `indexOf` or `filter` for path deduplication in nested directory traversal contexts.
- **R-DEDUP-007** MAY: Document exceptions with performance justification and benchmark data, referencing exception ID (e.g., EXC-001) in code comments.

### Verify

```bash
# Verify Set usage for path/key collection
grep -r 'new Set<' src/ tests/ | grep -E '(path|gitignore|seen|targets)' | wc -l

# Verify add() method calls on Sets
grep -r '\.add\(' src/ tests/ | grep -E '(Set|pathSet|seen|targets|expectedGitignoreEntries)' | wc -l

# Verify JSON.parse() patterns with file reading
grep -r 'JSON\.parse' src/ | grep -E '(fs\.readFile|existingContent|content)' | wc -l
```

**Accept when:**
- All three grep commands return non-zero counts, confirming Set usage, `add()` method calls, and `JSON.parse()` patterns exist in the codebase.
- Code review confirms no array-based path collection in nested directory traversal functions.
- Integration tests pass for nested MCP configuration generation without duplicate gitignore entries.
- No violations of R-DEDUP-006 are detected in code review.

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST return non-zero counts before accepting changes to path collection, gitignore generation, or configuration key aggregation logic. Code review MUST reject array-based deduplication patterns in scope. Integration tests MUST validate gitignore uniqueness across nested structures.
</enforcement>