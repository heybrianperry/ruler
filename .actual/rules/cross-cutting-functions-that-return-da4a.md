# Adopt Set-Based Message Queue Pattern for Deduplication in Agent Configuration: Functions That Return

These rules are ALWAYS ACTIVE for all agent configuration processing, MCP server management, skills propagation, and revert operations within the Ruler system, including FirebenderAgent, Claude, Copilot, Windsurf agent implementations, SkillsProcessor, revert-engine, and gitignore management modules.

### Rules

- **R-DEDUP-001** SHOULD: Functions that return deduplicated collections SHOULD convert Sets to Arrays only at API boundaries to preserve type safety.

### Verify

```bash
# Verify Set-based deduplication patterns across configuration modules
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
- Sets are converted to Arrays using Array.from(set) or [...set] only at function return boundaries or when passing to APIs that require arrays
- File paths are normalized (using path.resolve() or path.normalize()) before adding to Sets to prevent false uniqueness from path format variations

<enforcement>
Code review MUST verify Set usage for all new configuration accumulation logic. CI pipeline MUST check for Set instantiation patterns in configuration modules via grep. Integration tests MUST validate idempotent behavior in nested directory scenarios. Claude Code MUST NOT skip or defer verification of these rules.
</enforcement>