# Adopt Set-Based Message Queue Pattern for MCP Configuration Propagation: Agent Configuration Processors

These rules are ALWAYS ACTIVE for all agent configuration propagation workflows involving MCP server definitions, skill targets, and gitignore path management across nested directory structures.

### Rules

- **R-MCP-CONFIG-001** SHOULD: Agent configuration processors SHOULD implement loadExistingConfig, saveConfig, and handleMcpConfiguration functions to separate concerns of reading, writing, and transforming configuration state.
- **R-MCP-CONFIG-002** MUST: Use Set data structures for accumulating unique identifiers in configuration propagation: gitignore paths, MCP server keys, skill targets, and configuration file paths.
- **R-MCP-CONFIG-003** MUST: Convert Sets to arrays using Array.from() or spread operator before JSON serialization or file writing operations.
- **R-MCP-CONFIG-004** SHOULD: Sort converted arrays to ensure deterministic output ordering for version control and diff comparison.
- **R-MCP-CONFIG-005** SHOULD: Implement logging at debug level when Set.add() encounters an existing entry to aid troubleshooting.
- **R-MCP-CONFIG-006** MUST: Prevent duplicate entries in gitignore files, MCP server configurations, and skill targets across multiple propagation operations.
- **R-MCP-CONFIG-007** SHOULD: Provide test utility functions for Set comparison (e.g., setsEqual, setContainsAll) to simplify assertion of expected configuration state in tests.

### Verify

```bash
# Verify Set instantiations for configuration deduplication
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
- All configuration processors implement loadExistingConfig, saveConfig, and handleMcpConfiguration functions
- Converted arrays are sorted before writing to configuration files for deterministic output

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration tests MUST pass and code review MUST confirm Set-based deduplication is implemented across all configuration propagation workflows. CI pipeline MUST fail if duplicate entries are detected in generated configuration files.
</enforcement>