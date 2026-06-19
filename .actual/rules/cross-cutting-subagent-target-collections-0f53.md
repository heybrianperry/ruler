# Adopt Set-Based Collection Pattern for Subagent Target Management: Subagent Target Collections

These rules are ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections or alias mappings, including SubagentsProcessor, SubagentsUtils, and any module that aggregates subagent targets, tool aliases, or gitignore paths exposed through public API contracts.

### Rules

- **R-SUBAGENT-001** MUST: Subagent target collections MUST use Set data structures (targets.add(target)) to ensure automatic deduplication during aggregation.
- **R-SUBAGENT-002** MUST: Initialize target and alias collections as Set instances at the beginning of aggregation functions: `const targets = new Set<string>()`.
- **R-SUBAGENT-003** MUST: Use .add() method for all insertions during recursive discovery and parsing phases to leverage automatic deduplication.
- **R-SUBAGENT-004** MUST: Convert Set to Array at public API boundaries using `Array.from(set)` or `[...set]` spread syntax when consumers require array semantics.
- **R-SUBAGENT-005** SHOULD: Document in JSDoc comments that public API contracts return deduplicated collections and do not preserve discovery order.
- **R-SUBAGENT-006** SHOULD: Consider adding debug logging when duplicates are detected (set.has() check before add()) to aid troubleshooting without changing semantics.
- **R-SUBAGENT-007** SHOULD: Convert Sets to sorted arrays at public API boundaries (Array.from(set).sort()) to ensure deterministic behavior in order-dependent operations.

### Verify

```bash
# Verify Set-based collection operations in core modules
grep -r 'targets\.add\|mapped\.add' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify Set instantiation patterns
grep -r 'new Set<' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify test coverage for deduplication behavior
npm test -- --grep 'deduplication|unique' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'
```

**Accept when:**
- All target and alias collection operations in SubagentsProcessor.ts and SubagentsUtils.ts use Set.add() method
- Public API contracts (discoverSubagents, getSelectedSubagentTargets, getSubagentsGitignorePaths) return arrays derived from Set collections
- Test suite includes validation that duplicate targets/aliases from multiple sources result in single collection entries
- No Array-based deduplication patterns (filter/includes) are present in subagent processing modules

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST reject new Array-based deduplication logic in subagent processing modules. CI pipeline MUST validate that public API contracts return unique collections. Static analysis linting rules MUST detect and flag prohibited deduplication patterns.
</enforcement>