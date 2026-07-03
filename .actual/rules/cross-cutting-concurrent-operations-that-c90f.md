# Use Set-Based Collection for Message Queue Target Management: Concurrent Operations That

These rules are ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections for message queue boundaries, including the SubagentsProcessor and SubagentsUtils modules and all functions that accumulate targets or aliases for message queue boundaries.

### Rules

- **R-SETCOLL-001** MUST: Concurrent operations that populate target collections MUST NOT assume insertion order or rely on array indexing semantics.

### Verify

```bash
# Verify Set.add() patterns in message queue boundary code
grep -r '\.add(' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts | grep -E '(targets|mapped)'

# Verify Set initialization
grep -r 'new Set' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify tests for duplicate handling
npm test -- --grep 'SubagentsProcessor|SubagentsUtils' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'
```

**Accept when:**
- All message queue boundary operations in SubagentsProcessor and SubagentsUtils use Set.add() for target accumulation
- No array-based duplicate checking logic (includes, indexOf, filter) exists in target collection management code
- Tests verify that duplicate targets are automatically deduplicated without manual intervention
- Set collections are initialized at function scope with explicit type safety: `const targets = new Set<string>();`
- Serialization to JSON or external APIs uses Array.from(targetSet) or [...targetSet] for conversion

<enforcement>
Claude Code MUST NOT skip or defer verification. All message queue boundary operations must use Set-based collections. Code review must block merges introducing array-based collections without documented exception approval. CI pipeline must fail if grep verification commands do not find expected Set.add() patterns.
</enforcement>