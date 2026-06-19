# Use Set-Based Collection for Message Queue Target Management: Modules That Coordinate

These rules are ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections for message queue boundaries, specifically SubagentsProcessor and SubagentsUtils modules coordinating subagent discovery, selection, and propagation across the system.

### Rules

- **R-COORD-001** SHOULD: Modules that coordinate subagent processing SHOULD separate concerns between discovery (SubagentsProcessor) and utility operations (SubagentsUtils).
- **R-COORD-002** MUST: All functions that accumulate targets or aliases for message queue boundaries MUST use Set data structures (targets.add(target), mapped.add(alias)) to prevent duplicate entries.
- **R-COORD-003** MUST: Concurrent file system operations that discover or load subagent configurations MUST use Set-based collections to ensure thread-safe accumulation of unique identifiers.
- **R-COORD-004** SHOULD: Initialize Set collections at function scope with explicit type safety: const targets = new Set<string>();
- **R-COORD-005** SHOULD: Use Array.from(targetSet) or [...targetSet] for serialization to JSON or external API responses that require array format.
- **R-COORD-006** SHOULD: Convert Sets to sorted arrays at API boundaries where ordering matters; document that internal Set order is not guaranteed.
- **R-COORD-007** MUST: Implement explicit cleanup of Set collections in finally blocks to prevent memory leaks after processing large batches of subagent discoveries.
- **R-COORD-008** MUST: Enforce TypeScript generic Set<T> types with strict type checking; prevent Set<any> usage via linting rules.

### Verify

```bash
# Verify Set.add() patterns in message queue boundary code
grep -r '\.add(' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts | grep -E '(targets|mapped)'

# Verify Set initialization patterns
grep -r 'new Set' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts

# Verify tests for duplicate handling
npm test -- --grep 'SubagentsProcessor|SubagentsUtils' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'

# Verify no array-based duplicate checking in target collection code
grep -r -E '(includes|indexOf|filter)' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts | grep -v 'node_modules' | grep -v '.test.ts'
```

**Accept when:**
- All message queue boundary operations in SubagentsProcessor and SubagentsUtils use Set.add() for target accumulation
- No array-based duplicate checking logic (includes, indexOf, filter) exists in target collection management code
- Tests verify that duplicate targets are automatically deduplicated without manual intervention
- Set collections are initialized with explicit type parameters (Set<string>)
- Serialization to external APIs uses Array.from() or spread operator conversion
- Memory cleanup is implemented in finally blocks for long-running operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep verification commands MUST pass before accepting changes to SubagentsProcessor.ts or SubagentsUtils.ts. Code review MUST block merge if array-based collections are introduced without documented exception approval (EXC-001 or equivalent).
</enforcement>