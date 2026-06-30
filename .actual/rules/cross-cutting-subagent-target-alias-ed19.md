# Adopt Message Queue Pattern for Subagent Target Collection: Subagent Target Alias

These rules are ALWAYS ACTIVE for all subagent file parsing operations, discovery workflows, and target/alias aggregation functions in SubagentsUtils.ts and SubagentsProcessor.ts.

### Rules

- **R-SUBAGENT-001** MUST: Subagent target and alias collection MUST use Set-based accumulation patterns (e.g., `mapped.add(alias)`, `targets.add(target)`) to ensure deduplication across multiple file sources.

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
- All subagent collection operations in SubagentsUtils.ts and SubagentsProcessor.ts use `Set.add()` or equivalent deduplicating data structures
- Public API functions (getSelectedSubagentTargets, discoverSubagents) return deduplicated collections without requiring caller-side deduplication
- Unit tests verify that duplicate aliases from multiple files result in single entries in collected results
- No array accumulation with manual deduplication patterns are found in subagent collection contexts

<enforcement>
Clause Code MUST NOT skip or defer verification of Set-based collection patterns in subagent processing code. All new subagent collection functions MUST be reviewed for compliance with R-SUBAGENT-001 before merge.
</enforcement>