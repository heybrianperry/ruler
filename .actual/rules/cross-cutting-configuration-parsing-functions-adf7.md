# Adopt Set-Based Collection Pattern for Subagent Target Management: Configuration Parsing Functions

These rules are ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections or alias mappings, including SubagentsProcessor, SubagentsUtils, and any module that aggregates subagent targets, tool aliases, or gitignore paths exposed through public API contracts.

### Rules

- **R-SBCP-001** SHOULD: Configuration parsing functions (parseFrontmatter, loadSubagentFile) SHOULD validate uniqueness constraints at the point of collection insertion using Set-based data structures.

### Verify

```bash
# Verify Set-based collection operations are used in core modules
grep -r 'targets\.add\|mapped\.add' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify Set initialization in collection aggregation
grep -r 'new Set<' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify deduplication tests exist
npm test -- --grep 'deduplication|unique' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'
```

**Accept when:**
- All target and alias collection operations in SubagentsProcessor.ts and SubagentsUtils.ts use Set.add() method
- Public API contracts (discoverSubagents, getSelectedSubagentTargets, getSubagentsGitignorePaths) return arrays derived from Set collections
- Test suite includes validation that duplicate targets/aliases from multiple sources result in single collection entries
- Configuration parsing functions convert Set to Array at public API boundaries using Array.from(set) or spread syntax
- JSDoc comments document that public API contracts return deduplicated collections and do not preserve discovery order

<enforcement>
Claude Code MUST NOT skip or defer verification of Set-based collection patterns in subagent processing modules. All new subagent aggregation functions MUST use Set-based deduplication. Code review rejection is mandatory for Array-based deduplication logic in these modules.
</enforcement>