# Use Set-Based Collection for Message Queue Target Management: Target Accumulation Operations

These rules are ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections for message queue boundaries, specifically SubagentsProcessor and SubagentsUtils modules coordinating subagent discovery, selection, and propagation across the system.

### Rules

- **R-SETCOLL-001** MUST: Target accumulation operations MUST use the add() method pattern (e.g., targets.add(target), mapped.add(alias)) for insertion into Set-based collections.

### Verify

```bash
# Verify Set.add() patterns are used in target accumulation
grep -r '\.add(' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts | grep -E '(targets|mapped)'

# Verify Set collections are initialized
grep -r 'new Set' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify tests check for duplicate deduplication
npm test -- --grep 'SubagentsProcessor|SubagentsUtils' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'
```

**Accept when:**
- All message queue boundary operations in SubagentsProcessor and SubagentsUtils use Set.add() for target accumulation
- No array-based duplicate checking logic (includes, indexOf, filter) exists in target collection management code
- Tests verify that duplicate targets are automatically deduplicated without manual intervention
- Set collections are initialized with proper TypeScript generic types (e.g., new Set<string>())

<enforcement>
Claude Code MUST NOT skip or defer verification. All target accumulation operations in scope MUST use Set.add() pattern. Violations block merge and require documented exception approval from architecture review board.
</enforcement>