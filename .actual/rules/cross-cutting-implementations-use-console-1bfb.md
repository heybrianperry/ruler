# Adopt Set-Based Message Queue Pattern for MCP Configuration Propagation: Implementations Use Console

These rules are ALWAYS ACTIVE for all agent configuration propagation workflows involving MCP server definitions, skill targets, and gitignore path management across nested directory structures.

### Rules

- **R-MCP-CONSOLE-001** MAY: Implementations MAY use console.warn for logging parse failures when reading existing configuration files.

### Verify

```bash
# Verify Set-based deduplication patterns in configuration processors
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
Claude Code MUST NOT skip or defer verification. Integration tests must pass and static analysis must confirm Set-based deduplication patterns are in place before accepting configuration propagation implementations.
</enforcement>