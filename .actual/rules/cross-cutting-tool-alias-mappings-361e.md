# Adopt Set-Based Collection Pattern for Subagent Target Management: Tool Alias Mappings

These rules are ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections or alias mappings, including SubagentsProcessor, SubagentsUtils, and any module that aggregates subagent targets, tool aliases, or gitignore paths.

### Rules

- **R-ALIAS-001** MUST: Tool alias mappings MUST use Set data structures (mapped.add(alias)) to prevent duplicate alias registration.

### Verify

```bash
# Verify Set-based collection operations in core modules
grep -r 'targets\.add\|mapped\.add' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify Set initialization in collection functions
grep -r 'new Set<' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify deduplication tests pass
npm test -- --grep 'deduplication|unique' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'
```

**Accept when:**
- All target and alias collection operations in SubagentsProcessor.ts and SubagentsUtils.ts use Set.add() method
- Public API contracts (discoverSubagents, getSelectedSubagentTargets, getSubagentsGitignorePaths) return arrays derived from Set collections
- Test suite includes validation that duplicate targets/aliases from multiple sources result in single collection entries

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST reject Array-based deduplication patterns in subagent processing modules. CI pipeline MUST validate that public API contracts return unique collections.
</enforcement>