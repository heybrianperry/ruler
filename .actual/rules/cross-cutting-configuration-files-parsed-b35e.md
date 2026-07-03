# Adopt Set-Based Message Queue Pattern for MCP Configuration Propagation: Configuration Files Parsed

These rules are ALWAYS ACTIVE for all agent configuration propagation workflows involving MCP server definitions, skill targets, and gitignore path management across nested directory structures.

### Rules

- **R-CONFIG-001** MUST: Configuration files MUST be parsed using JSON.parse before extracting existing server definitions or configuration state.

### Verify

```bash
# Verify Set-based deduplication patterns in configuration processors
grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|targets|seen|servers)' | wc -l

# Verify Set.add() operations across configuration aggregation workflows
grep -r '\.add\(' src/ tests/ | grep -E '(expectedGitignoreEntries|targets|seen)' | wc -l

# Run nested MCP propagation integration tests
npm test -- --testNamePattern='Nested MCP propagation' --verbose
```

**Accept when:**
- Grep commands identify at least 3 distinct Set instantiations and 5+ Set.add() operations across configuration processors
- Nested MCP propagation integration test passes, verifying no duplicate entries in gitignore or MCP configuration files
- Code review confirms all configuration aggregation workflows use Set-based deduplication before writing output
- JSON.parse is called on all configuration file contents before extracting server definitions or state

<enforcement>
Clause R-CONFIG-001 is mandatory. Claude Code MUST verify that configuration files are parsed with JSON.parse before any extraction or state operations. Integration tests MUST pass before accepting configuration propagation changes. Code review MUST confirm Set-based deduplication is used in all configuration processors.
</enforcement>