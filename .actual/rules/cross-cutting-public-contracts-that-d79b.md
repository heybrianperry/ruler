# Adopt Set-Based Collection Pattern for Subagent Target Management: Public Contracts That

These rules are ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections or alias mappings, including the SubagentsProcessor module (src/core/SubagentsProcessor.ts), SubagentsUtils module (src/core/SubagentsUtils.ts), and any module that aggregates subagent targets, tool aliases, or gitignore paths exposed as public API contracts.

### Rules

- **R-SUBAGENT-001** MUST: Public API contracts that return subagent collections (discoverSubagents, getSelectedSubagentTargets, getSubagentsGitignorePaths) MUST guarantee uniqueness of returned elements through Set-based collection semantics.

- **R-SUBAGENT-002** MUST: Initialize target and alias collections as Set instances at the beginning of aggregation functions using `const targets = new Set<string>()`.

- **R-SUBAGENT-003** MUST: Use `.add()` method for all insertions during recursive discovery and parsing phases to leverage automatic deduplication.

- **R-SUBAGENT-004** MUST: Convert Set to Array at public API boundaries using `Array.from(set)` or `[...set]` spread syntax when consumers require array semantics.

- **R-SUBAGENT-005** SHOULD: Document in JSDoc comments that public API contracts return deduplicated collections and do not preserve discovery order.

- **R-SUBAGENT-006** SHOULD: Consider adding debug logging when duplicates are detected (set.has() check before add()) to aid troubleshooting without changing semantics.

- **R-SUBAGENT-007** SHOULD: Convert Sets to sorted arrays at public API boundaries using `Array.from(set).sort()` to ensure deterministic iteration order for testing and debugging.

### Verify

```bash
# Verify Set-based collection operations are used
grep -r 'targets\.add\|mapped\.add' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify Set initialization patterns
grep -r 'new Set<' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify test suite includes deduplication validation
npm test -- --grep 'deduplication|unique' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'
```

**Accept when:**
- All target and alias collection operations in SubagentsProcessor.ts and SubagentsUtils.ts use Set.add() method
- Public API contracts (discoverSubagents, getSelectedSubagentTargets, getSubagentsGitignorePaths) return arrays derived from Set collections
- Test suite includes validation that duplicate targets/aliases from multiple sources result in single collection entries
- No Array-based deduplication patterns (filter/includes) are present in subagent processing modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All Set-based collection patterns MUST be verified before accepting changes to subagent processing modules. Violations detected by static analysis or CI pipeline MUST result in code review rejection.
</enforcement>