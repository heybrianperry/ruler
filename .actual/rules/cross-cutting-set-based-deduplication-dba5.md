# Adopt Set-Based Message Queue Pattern for MCP Configuration Propagation: Set Based Deduplication

These rules are ALWAYS ACTIVE for all agent configuration propagation workflows involving MCP server definitions, skill targets, and gitignore path management across nested directory structures.

### Rules

- **R-SETDEDUP-001** SHOULD: Set-based deduplication SHOULD occur before writing final configuration to prevent accumulation of duplicate entries across multiple invocations.
- **R-SETDEDUP-002** MUST: Initialize Set instances at the beginning of configuration propagation workflows before iterating over agents or directories.
- **R-SETDEDUP-003** MUST: Use Set.add() for all operations that accumulate unique identifiers: gitignore paths, MCP server keys, skill targets, and configuration file paths.
- **R-SETDEDUP-004** MUST: Convert Sets to arrays using Array.from() or spread operator before JSON serialization or file writing operations.
- **R-SETDEDUP-005** SHOULD: Sort converted arrays to ensure deterministic output ordering for version control and diff comparison.
- **R-SETDEDUP-006** SHOULD: Implement helper functions like extractServers() to normalize configuration structure before extracting keys for Set-based deduplication.
- **R-SETDEDUP-007** SHOULD: Implement logging at debug level when Set.add() encounters an existing entry to aid troubleshooting.
- **R-SETDEDUP-008** SHOULD: Add integration tests that verify expected Set sizes after propagation.

### Verify

```bash
# Verify Set instantiations in configuration processors
grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|targets|seen|servers)' | wc -l

# Verify Set.add() operations across configuration aggregation
grep -r '\.add\(' src/ tests/ | grep -E '(expectedGitignoreEntries|targets|seen)' | wc -l

# Run nested MCP propagation integration test
npm test -- --testNamePattern='Nested MCP propagation' --verbose
```

**Accept when:**
- Grep commands identify at least 3 distinct Set instantiations and 5+ Set.add() operations across configuration processors
- Nested MCP propagation integration test passes, verifying no duplicate entries in gitignore or MCP configuration files
- Code review confirms all configuration aggregation workflows use Set-based deduplication before writing output

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration tests MUST pass and grep commands MUST confirm Set-based deduplication patterns are present before accepting configuration propagation changes.
</enforcement>