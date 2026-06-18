# Adopt Set-Based Message Queue Pattern for MCP Configuration Propagation: Set Based Accumulators

These rules are ALWAYS ACTIVE for all agent configuration propagation workflows involving MCP server definitions, skill targets, and gitignore path management across nested directory structures.

### Rules

- **R-SETACC-001** MUST: Set-based accumulators MUST be used when processing gitignore entries (expectedGitignoreEntries.add), MCP server keys (seen.add), and skill targets (targets.add).
- **R-SETACC-002** MUST: Initialize Set instances at the beginning of configuration propagation workflows before iterating over agents or directories.
- **R-SETACC-003** MUST: Convert Sets to arrays using Array.from() or spread operator before JSON serialization or file writing operations.
- **R-SETACC-004** SHOULD: Sort converted arrays to ensure deterministic output ordering for version control and diff comparison.
- **R-SETACC-005** SHOULD: Implement helper functions like extractServers() to normalize configuration structure before extracting keys for Set-based deduplication.
- **R-SETACC-006** SHOULD: Implement logging at debug level when Set.add() encounters an existing entry to aid troubleshooting.
- **R-SETACC-007** SHOULD: Provide test utility functions for Set comparison (e.g., setsEqual, setContainsAll) and use Array.from() with sorting for snapshot testing.

### Verify

```bash
# Verify Set instantiations for configuration accumulators
grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|targets|seen|servers)' | wc -l

# Verify Set.add() operations across configuration processors
grep -r '\.add\(' src/ tests/ | grep -E '(expectedGitignoreEntries|targets|seen)' | wc -l

# Run nested MCP propagation integration test
npm test -- --testNamePattern='Nested MCP propagation' --verbose
```

**Accept when:**
- Grep commands identify at least 3 distinct Set instantiations and 5+ Set.add() operations across configuration processors
- Nested MCP propagation integration test passes, verifying no duplicate entries in gitignore or MCP configuration files
- Code review confirms all configuration aggregation workflows use Set-based deduplication before writing output

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration propagation code paths must be reviewed to confirm Set-based deduplication is implemented before duplicate entries can accumulate in generated configuration files.
</enforcement>