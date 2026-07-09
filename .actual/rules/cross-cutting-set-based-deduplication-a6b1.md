# Adopt Set-Based Message Queue Pattern for Deduplication in Agent Configuration: Set Based Deduplication

These rules are ALWAYS ACTIVE for all agent configuration processing, MCP server management, skills propagation, and revert operations within the Ruler system, including agent configuration processors (FirebenderAgent, Claude, Copilot, Windsurf), MCP server configuration management, skills discovery and propagation, configuration revert operations, and gitignore path accumulation.

### Rules

- **R-DEDUP-001** MUST: Set-based deduplication MUST be applied before writing to gitignore files, MCP configuration files, or skill manifest files to ensure idempotent operations.
- **R-DEDUP-002** MUST: Initialize Set instances at the beginning of processing functions (e.g., `const pathSet = new Set<string>()`) to establish clear deduplication scope.
- **R-DEDUP-003** MUST: Use Set.add() immediately upon discovering paths, identifiers, or entries rather than batching additions to minimize duplicate processing windows.
- **R-DEDUP-004** MUST: For file path deduplication, ensure paths are normalized (e.g., using path.resolve() or path.normalize()) before adding to Sets to prevent false uniqueness from path format variations.
- **R-DEDUP-005** SHOULD: Convert Sets to Arrays using Array.from(set) or [...set] only at function return boundaries or when passing to APIs that require arrays.
- **R-DEDUP-006** SHOULD: Implement validation layer that detects and warns on duplicate keys with conflicting values before deduplication; log deduplicated items at debug level.
- **R-DEDUP-007** MAY: Use Map<string, boolean> to track seen items with explicit true values only when additional metadata must be associated with each unique item (e.g., timestamps, counts).

### Verify

```bash
# Count Set instantiations in configuration modules
grep -r 'new Set<' src/ tests/ | grep -E '(pathSet|seen|targets|processedPaths|expectedGitignoreEntries)' | wc -l

# Count Set.add() patterns in configuration modules
grep -r '\.add\(' src/ tests/ | grep -E '(pathSet|seen|targets|processedPaths|expectedGitignoreEntries)' | wc -l

# Run nested MCP propagation tests
npm test -- --testNamePattern='nested.*MCP.*propagation' --verbose
```

**Accept when:**
- All agent configuration processors use Set data structures for accumulating paths, identifiers, and gitignore entries
- Verification commands show Set.add() patterns in SkillsProcessor, FirebenderAgent, revert-engine, and MCP configuration modules
- Integration tests for nested MCP propagation pass, confirming idempotent gitignore and configuration file writes
- No Array-based deduplication patterns are found in configuration accumulation logic
- All file paths are normalized before being added to Sets

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST reject PRs introducing Array-based deduplication in configuration processing. CI pipeline MUST warn when new agent implementations lack Set-based patterns. Violations discovered in production code require post-merge remediation tickets.
</enforcement>