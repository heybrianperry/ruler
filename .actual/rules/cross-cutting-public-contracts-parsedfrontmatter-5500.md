# Adopt Message Queue Pattern for Subagent Target Collection: Public Contracts Parsedfrontmatter

These rules are ALWAYS ACTIVE for all subagent file parsing operations in SubagentsUtils.ts, all subagent discovery workflows in SubagentsProcessor.ts, and all functions that aggregate targets, aliases, or mappings from multiple sources.

### Rules

- **R-MSGQ-001** SHOULD: Public API contracts (ParsedFrontmatter, CopilotToolMapping, getSelectedSubagentTargets) SHOULD expose collected results as immutable data structures.
- **R-MSGQ-002** MUST: Initialize Set collections at the start of discovery functions (discoverSubagents, loadSubagentFile) before entering async loops.
- **R-MSGQ-003** MUST: Use descriptive variable names (targets, aliases, mapped) that clearly indicate the collection is accumulating deduplicated results.
- **R-MSGQ-004** MUST: Convert Sets to Arrays using Array.from() or spread operator before returning from public API functions if consumers expect iterable collections.
- **R-MSGQ-005** SHOULD: Add debug logging that reports collection size at key checkpoints (after parsing each file, after completing discovery) to aid troubleshooting.
- **R-MSGQ-006** MUST: Ensure all subagent collection operations use Set.add() or equivalent deduplicating data structures, not Array.push() with manual deduplication.
- **R-MSGQ-007** SHOULD: Document that collection order is not guaranteed; convert Sets to sorted arrays before propagation if deterministic ordering is required.
- **R-MSGQ-008** SHOULD: Add optional logging or warning mode that reports when duplicate aliases are encountered during collection, configurable via environment variable.

### Verify

```bash
# Verify Set-based collection patterns are used
grep -r 'mapped\.add\|targets\.add' src/core/Subagents*.ts

# Count Set instantiations in subagent processing code
grep -r 'new Set<' src/core/Subagents*.ts | wc -l

# Run subagent-specific unit tests with verbose output
npm test -- --testPathPattern='Subagents' --verbose
```

**Accept when:**
- All subagent collection operations in SubagentsUtils.ts and SubagentsProcessor.ts use Set.add() or equivalent deduplicating data structures
- Public API functions (getSelectedSubagentTargets, discoverSubagents) return deduplicated collections without requiring caller-side deduplication
- Unit tests verify that duplicate aliases from multiple files result in single entries in collected results
- No Array.push() patterns are found in subagent collection contexts
- Deduplication tests pass after changes to SubagentsUtils or SubagentsProcessor

<enforcement>
Code review MUST verify Set-based collection for new subagent processing functions. Unit tests MUST verify deduplication behavior when processing multiple subagent files with overlapping aliases. Static analysis or linting MUST flag Array.push() patterns in subagent collection contexts. CI test failures for deduplication tests MUST block merge. Architecture review is REQUIRED for any PR introducing alternative collection patterns in subagent processing code. Claude Code MUST NOT skip or defer verification of these rules.
</enforcement>