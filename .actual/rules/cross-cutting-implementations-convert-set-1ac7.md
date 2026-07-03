# Use Set-Based Collection for Message Queue Target Management: Implementations Convert Set

These rules are ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections for message queue boundaries, specifically SubagentsProcessor and SubagentsUtils modules coordinating subagent discovery, selection, and propagation across the system.

### Rules

- **R-SETCOLL-001** MAY: Implementations MAY convert Set collections to arrays for serialization or external API boundaries.

### Verify

```bash
# Verify Set.add() patterns in message queue boundary code
grep -r '\.add(' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts | grep -E '(targets|mapped)'

# Verify Set initialization patterns
grep -r 'new Set' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify duplicate handling tests exist
npm test -- --grep 'SubagentsProcessor|SubagentsUtils' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'
```

**Accept when:**
- All message queue boundary operations in SubagentsProcessor and SubagentsUtils use Set.add() for target accumulation
- No array-based duplicate checking logic (includes, indexOf, filter) exists in target collection management code
- Tests verify that duplicate targets are automatically deduplicated without manual intervention
- Set collections are converted to arrays using Array.from() or spread operator [...set] at serialization boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merges if array-based collections are introduced without documented exception approval. CI pipeline MUST fail if grep verification commands do not find expected Set.add() patterns in SubagentsProcessor and SubagentsUtils.
</enforcement>