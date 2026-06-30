# Adopt Message Queue Pattern for Subagent Target Collection: Collection Operations Occur

These rules are ALWAYS ACTIVE for all subagent file parsing operations, discovery workflows, and functions that aggregate targets, aliases, or mappings from multiple sources in SubagentsUtils.ts and SubagentsProcessor.ts.

### Rules

- **R-MSGQ-001** MUST: Collection operations MUST occur during file parsing and discovery phases before propagation to downstream consumers.
- **R-MSGQ-002** MUST: Initialize Set collections at the start of discovery functions (discoverSubagents, loadSubagentFile) before entering async loops.
- **R-MSGQ-003** MUST: Use Set.add() or equivalent deduplicating data structures for accumulating targets, aliases, and mappings across multiple file sources.
- **R-MSGQ-004** SHOULD: Use descriptive variable names (targets, aliases, mapped) that clearly indicate the collection is accumulating deduplicated results.
- **R-MSGQ-005** SHOULD: Convert Sets to Arrays using Array.from() or spread operator before returning from public API functions if consumers expect iterable collections.
- **R-MSGQ-006** SHOULD: Add debug logging that reports collection size at key checkpoints (after parsing each file, after completing discovery) to aid troubleshooting.
- **R-MSGQ-007** MAY: Add optional logging or warning mode that reports when duplicate aliases are encountered during collection, configurable via environment variable.

### Verify

```bash
# Verify Set-based collection patterns are used
grep -r 'mapped\.add\|targets\.add' src/core/Subagents*.ts

# Count Set instantiations in subagent modules
grep -r 'new Set<' src/core/Subagents*.ts | wc -l

# Run subagent-specific tests with verbose output
npm test -- --testPathPattern='Subagents' --verbose
```

**Accept when:**
- All subagent collection operations in SubagentsUtils.ts and SubagentsProcessor.ts use Set.add() or equivalent deduplicating data structures
- Public API functions (getSelectedSubagentTargets, discoverSubagents) return deduplicated collections without requiring caller-side deduplication
- Unit tests verify that duplicate aliases from multiple files result in single entries in collected results
- No Array.push() patterns are found in subagent collection contexts
- Collection size is logged at key checkpoints during discovery operations

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. Code review checklist items, unit tests, and static analysis must confirm Set-based collection patterns before merging changes to SubagentsUtils.ts or SubagentsProcessor.ts. Architecture review is required for any PR introducing alternative collection patterns in subagent processing code.
</enforcement>