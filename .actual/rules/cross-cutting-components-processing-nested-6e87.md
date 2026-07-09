# Adopt Set-Based Message Queue Pattern for Deduplication in Agent Configuration: Components Processing Nested

These rules are ALWAYS ACTIVE for all agent configuration processing, MCP server management, skills propagation, and revert operations within the Ruler system, including agent configuration processors (FirebenderAgent, Claude, Copilot, Windsurf), MCP server configuration management, skills discovery and propagation, configuration revert operations, and gitignore path accumulation.

### Rules

- **R-DEDUP-001** MUST: Components processing nested directory hierarchies MUST maintain separate Set instances per processing context to isolate deduplication scope.
- **R-DEDUP-002** MUST: Initialize Set instances at the beginning of processing functions (e.g., `const pathSet = new Set<string>()`) to establish clear deduplication scope.
- **R-DEDUP-003** MUST: Use Set.add() immediately upon discovering paths, identifiers, or entries rather than batching additions to minimize duplicate processing windows.
- **R-DEDUP-004** MUST: Convert Sets to Arrays using `Array.from(set)` or `[...set]` only at function return boundaries or when passing to APIs that require arrays.
- **R-DEDUP-005** MUST: For file path deduplication, ensure paths are normalized (e.g., using `path.resolve()` or `path.normalize()`) before adding to Sets to prevent false uniqueness from path format variations.
- **R-DEDUP-006** SHOULD: Implement validation layer that detects and warns on duplicate keys with conflicting values before deduplication; log deduplicated items at debug level.
- **R-DEDUP-007** SHOULD: Use TypeScript readonly arrays to signal immutability expectations at API boundaries where deduplication occurs.

### Verify

```bash
# Count Set instantiations for deduplication variables
grep -r 'new Set<' src/ tests/ | grep -E '(pathSet|seen|targets|processedPaths|expectedGitignoreEntries)' | wc -l

# Count Set.add() calls on deduplication variables
grep -r '\.add\(' src/ tests/ | grep -E '(pathSet|seen|targets|processedPaths|expectedGitignoreEntries)' | wc -l

# Run nested MCP propagation tests
npm test -- --testNamePattern='nested.*MCP.*propagation' --verbose
```

**Accept when:**
- All agent configuration processors use Set data structures for accumulating paths, identifiers, and gitignore entries
- Verification commands show Set.add() patterns in SkillsProcessor, FirebenderAgent, revert-engine, and MCP configuration modules
- Integration tests for nested MCP propagation pass, confirming idempotent gitignore and configuration file writes
- No Array-based deduplication patterns exist in configuration accumulation logic
- All file paths are normalized before Set insertion

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for agent configuration processing code. Violations must be caught during code review and CI pipeline checks before merge.
</enforcement>