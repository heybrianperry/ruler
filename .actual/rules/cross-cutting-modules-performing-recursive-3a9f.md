# Adopt Set-Based Collection Pattern for Subagent Target Management: Modules Performing Recursive

These rules are ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections or alias mappings, including recursive filesystem discovery and multi-source aggregation operations.

### Rules

- **R-SUBAGENT-001** SHOULD: Modules performing recursive filesystem operations (listMarkdownFilesRecursive) SHOULD accumulate results into Set-based collections before conversion to arrays.

### Verify

```bash
# Verify Set-based collection operations in core modules
grep -r 'targets\.add\|mapped\.add' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify Set instantiation in subagent processing
grep -r 'new Set<' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify test suite includes deduplication validation
npm test -- --grep 'deduplication|unique' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'
```

**Accept when:**
- All target and alias collection operations in SubagentsProcessor.ts and SubagentsUtils.ts use Set.add() method
- Public API contracts (discoverSubagents, getSelectedSubagentTargets, getSubagentsGitignorePaths) return arrays derived from Set collections
- Test suite includes validation that duplicate targets/aliases from multiple sources result in single collection entries

<enforcement>
Code review MUST verify Set-based collections for all new subagent aggregation functions. Static analysis linting rules MUST detect Array-based deduplication patterns (filter/includes) in subagent processing modules. CI pipeline test suite MUST validate that public API contracts return unique collections. Claude Code MUST NOT skip or defer verification of these rules.
</enforcement>