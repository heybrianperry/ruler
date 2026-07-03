# Adopt Message Queue Pattern for Subagent Target Collection: Subagent Target Collection

These rules are ALWAYS ACTIVE for all subagent discovery and processing workflows, including SubagentsUtils.ts, SubagentsProcessor.ts, Copilot tool mapping, Claude agent propagation, and markdown file discovery operations.

### Rules

- **R-SUBAGENT-001** MUST: Subagent target collection MUST use Set-based data structures to ensure uniqueness of discovered targets.

### Verify

```bash
# Verify Set-based collection usage in subagent discovery modules
grep -r '\.add(' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts | grep -E '(mapped|targets)'

# Verify Set type declarations in subagent code
grep -r 'Set<' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts

# Run subagent collection and uniqueness tests
npm test -- --testPathPattern='Subagents(Utils|Processor)' --testNamePattern='collection|uniqueness'
```

**Accept when:**
- All subagent discovery operations use Set-based collections with .add() accumulation patterns
- No duplicate subagent targets appear in propagated collections to external tools (Copilot, Claude)
- Tests verify uniqueness guarantees and proper accumulation across multiple discovery invocations
- Discovery functions (discoverSubagents, loadSubagentFile) accept collection references as parameters

<enforcement>
Clause MUST NOT skip or defer verification. Code review must confirm Set-based collection patterns before merge. Pull requests introducing array-based collections without uniqueness guarantees must be revised. Failed uniqueness tests block merge until collection patterns are corrected.
</enforcement>