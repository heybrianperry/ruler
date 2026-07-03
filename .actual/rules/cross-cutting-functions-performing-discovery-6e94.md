# Adopt Message Queue Pattern for Subagent Target Collection: Functions Performing Discovery

These rules are ALWAYS ACTIVE for all subagent file parsing operations in SubagentsUtils.ts, all subagent discovery workflows in SubagentsProcessor.ts, functions that aggregate targets, aliases, or mappings from multiple sources, and asynchronous file system operations that accumulate results incrementally.

### Rules

- **R-MSGQ-001** SHOULD: Functions performing discovery (loadSubagentFile, discoverSubagents, listMarkdownFilesRecursive) SHOULD separate collection logic from validation and transformation logic.
- **R-MSGQ-002** SHOULD: Initialize Set collections at the start of discovery functions before entering async loops.
- **R-MSGQ-003** SHOULD: Use descriptive variable names (targets, aliases, mapped) that clearly indicate the collection is accumulating deduplicated results.
- **R-MSGQ-004** SHOULD: Convert Sets to Arrays using Array.from() or spread operator before returning from public API functions if consumers expect iterable collections.
- **R-MSGQ-005** MAY: Consider adding debug logging that reports collection size at key checkpoints (after parsing each file, after completing discovery) to aid troubleshooting.

### Verify

```bash
# Verify Set-based collection patterns are used
grep -r 'mapped\.add\|targets\.add' src/core/Subagents*.ts

# Count Set instantiations in subagent modules
grep -r 'new Set<' src/core/Subagents*.ts | wc -l

# Run subagent-related tests with verbose output
npm test -- --testPathPattern='Subagents' --verbose
```

**Accept when:**
- All subagent collection operations in SubagentsUtils.ts and SubagentsProcessor.ts use Set.add() or equivalent deduplicating data structures
- Public API functions (getSelectedSubagentTargets, discoverSubagents) return deduplicated collections without requiring caller-side deduplication
- Unit tests verify that duplicate aliases from multiple files result in single entries in collected results

<enforcement>
Code review MUST verify Set-based collection for new subagent processing functions. Unit tests MUST verify deduplication behavior when processing multiple subagent files with overlapping aliases. Static analysis or linting rules SHOULD flag Array.push() patterns in subagent collection contexts. Violations require refactor to Set-based collection or architecture review approval. CI test failures on deduplication tests block merge. Alternative collection structures require documented rationale in code comments and tech lead approval.
</enforcement>