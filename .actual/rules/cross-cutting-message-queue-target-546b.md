# Use Set-Based Collection for Message Queue Target Management: Message Queue Target

These rules are ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections for message queue boundaries, specifically SubagentsProcessor and SubagentsUtils modules and all functions that accumulate targets or aliases for message queue boundaries.

### Rules

- **R-MQT-001** MUST: All message queue target collections MUST use Set data structures to ensure uniqueness of targets and aliases.

### Verify

```bash
# Verify Set.add() patterns in target collection code
grep -r '\.add(' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts | grep -E '(targets|mapped)'

# Verify Set initialization
grep -r 'new Set' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify no array-based duplicate checking in target management
grep -r 'includes\|indexOf\|filter' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts | grep -v 'node_modules' || echo 'No array-based duplicate checking found'

# Verify tests for duplicate handling
npm test -- --grep 'SubagentsProcessor|SubagentsUtils' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'
```

**Accept when:**
- All message queue boundary operations in SubagentsProcessor and SubagentsUtils use Set.add() for target accumulation
- No array-based duplicate checking logic (includes, indexOf, filter) exists in target collection management code
- Tests verify that duplicate targets are automatically deduplicated without manual intervention
- Set collections are initialized with explicit type safety: `const targets = new Set<string>();`
- Serialization to JSON or external APIs uses Array.from() or spread operator conversion

<enforcement>
Clause Code MUST NOT skip or defer verification. All message queue target collections in scope MUST use Set data structures. Violations block merge and require documented exception approval from architecture review board.
</enforcement>