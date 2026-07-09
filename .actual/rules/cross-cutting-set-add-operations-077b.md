# Adopt Set-Based Message Queue Pattern for Deduplication in Agent Configuration: Set Add Operations

These rules are ALWAYS ACTIVE for all agent configuration processing, MCP server management, skills propagation, and revert operations within the Ruler system, including FirebenderAgent, Claude, Copilot, Windsurf agent implementations, SkillsProcessor, revert-engine, and gitignore management modules.

### Rules

- **R-SETADD-001** SHOULD: Set.add() operations SHOULD occur immediately upon path or identifier discovery to minimize the window for duplicate processing.

### Verify

```bash
# Verify Set instantiation patterns in configuration modules
grep -r 'new Set<' src/ tests/ | grep -E '(pathSet|seen|targets|processedPaths|expectedGitignoreEntries)' | wc -l

# Verify Set.add() usage in deduplication contexts
grep -r '\.add\(' src/ tests/ | grep -E '(pathSet|seen|targets|processedPaths|expectedGitignoreEntries)' | wc -l

# Run nested MCP propagation integration tests
npm test -- --testNamePattern='nested.*MCP.*propagation' --verbose
```

**Accept when:**
- All agent configuration processors (FirebenderAgent, Claude, Copilot, Windsurf) use Set data structures for accumulating paths, identifiers, and gitignore entries
- Verification commands show Set.add() patterns present in SkillsProcessor, FirebenderAgent, revert-engine, and MCP configuration modules
- Integration tests for nested MCP propagation pass, confirming idempotent gitignore and configuration file writes
- No Array-based manual deduplication logic exists in configuration accumulation paths

<enforcement>
Clause Code MUST NOT skip or defer verification. All new agent configuration processors and configuration accumulation logic MUST use Set-based deduplication. Code review MUST reject PRs introducing Array-based deduplication in configuration processing. CI pipeline MUST warn when new agent implementations lack Set-based patterns.
</enforcement>