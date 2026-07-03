# Use Set-Based Collection for Message Queue Target Management: Public Contracts That

These rules are ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections for message queue boundaries, specifically SubagentsProcessor and SubagentsUtils modules coordinating subagent discovery, selection, and propagation across the system.

### Rules

- **R-SETCOLL-001** SHOULD: Public contracts that expose target collections SHOULD return immutable views or copies to prevent external mutation.

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
- Public API methods return immutable views or defensive copies of Set collections

<enforcement>
Clause Code MUST NOT skip or defer verification. Violations block CI builds and code review merges. Exceptions require two senior engineer sign-offs with documented re-evaluation criteria and EXC-ID references.
</enforcement>