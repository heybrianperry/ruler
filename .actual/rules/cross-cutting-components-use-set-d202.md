# Adopt Set-Based Message Queue Pattern for Deduplication in Agent Configuration: Components Use Set

These rules are ALWAYS ACTIVE for all agent configuration processing, MCP server management, skills propagation, and revert operations within the Ruler system, including FirebenderAgent, Claude, Copilot, Windsurf agent implementations, SkillsProcessor, revert-engine, and gitignore management modules.

### Rules

- **R-DEDUP-001** MAY: Components MAY use Set.has() checks before expensive file system operations to short-circuit redundant processing.
- **R-DEDUP-002** SHOULD: Initialize Set instances at the beginning of processing functions (e.g., `const pathSet = new Set<string>()`) to establish clear deduplication scope.
- **R-DEDUP-003** SHOULD: Use Set.add() immediately upon discovering paths, identifiers, or entries rather than batching additions to minimize duplicate processing windows.
- **R-DEDUP-004** SHOULD: Convert Sets to Arrays using `Array.from(set)` or `[...set]` only at function return boundaries or when passing to APIs that require arrays.
- **R-DEDUP-005** SHOULD: For file path deduplication, ensure paths are normalized (e.g., using `path.resolve()` or `path.normalize()`) before adding to Sets to prevent false uniqueness from path format variations.
- **R-DEDUP-006** MUST NOT: Use Array.includes() or Array.find() for manual deduplication in configuration accumulation paths where Set performance is available.
- **R-DEDUP-007** MUST NOT: Introduce Array-based deduplication in new agent configuration processors when Set-based patterns are established.

### Verify

```bash
# Count Set instantiations in configuration modules
grep -r 'new Set<' src/ tests/ | grep -E '(pathSet|seen|targets|processedPaths|expectedGitignoreEntries)' | wc -l

# Count Set.add() patterns in configuration modules
grep -r '\.add\(' src/ tests/ | grep -E '(pathSet|seen|targets|processedPaths|expectedGitignoreEntries)' | wc -l

# Run nested MCP propagation integration tests
npm test -- --testNamePattern='nested.*MCP.*propagation' --verbose
```

**Accept when:**
- All agent configuration processors (FirebenderAgent, Claude, Copilot, Windsurf) use Set data structures for accumulating paths, identifiers, and gitignore entries
- Verification commands show Set.add() patterns in SkillsProcessor, FirebenderAgent, revert-engine, and MCP configuration modules
- Integration tests for nested MCP propagation pass, confirming idempotent gitignore and configuration file writes
- No Array-based deduplication patterns exist in configuration accumulation logic

<enforcement>
Clause Code MUST NOT skip or defer verification. All new agent configuration processors and configuration accumulation logic MUST conform to R-DEDUP-001 through R-DEDUP-007. Code review MUST reject PRs introducing Array-based deduplication in configuration processing. Violations discovered post-merge require remediation tickets.
</enforcement>