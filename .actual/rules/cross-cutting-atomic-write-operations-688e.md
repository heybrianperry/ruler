# Adopt Set-Based Collection Pattern for Subagent Target Management: Atomic Write Operations

These rules are ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections or alias mappings, including SubagentsProcessor (src/core/SubagentsProcessor.ts), SubagentsUtils (src/core/SubagentsUtils.ts), and any module that aggregates subagent targets, tool aliases, or gitignore paths.

### Rules

- **R-ATOMIC-001** MAY: Atomic write operations (writeAgentsDirectoryAtomic) MAY convert Set collections to arrays for serialization while preserving uniqueness guarantees.

### Verify

```bash
# Verify Set-based collection operations are used
grep -r 'targets\.add\|mapped\.add' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify Set initialization patterns
grep -r 'new Set<' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify deduplication tests exist
npm test -- --grep 'deduplication|unique' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'
```

**Accept when:**
- All target and alias collection operations in SubagentsProcessor.ts and SubagentsUtils.ts use Set.add() method
- Public API contracts (discoverSubagents, getSelectedSubagentTargets, getSubagentsGitignorePaths) return arrays derived from Set collections
- Test suite includes validation that duplicate targets/aliases from multiple sources result in single collection entries
- Conversion between Set and Array representations occurs at public API boundaries using Array.from(set) or spread syntax
- JSDoc comments document that public API contracts return deduplicated collections and do not preserve discovery order

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST reject new Array-based deduplication logic in subagent processing modules. CI pipeline MUST validate that public API contracts return unique collections. Static analysis linting rules MUST detect and flag Array-based deduplication patterns (filter/includes) in subagent processing modules.
</enforcement>