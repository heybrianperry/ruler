# Adopt Message Queue Pattern for Subagent Target Collection: Collection Structures Passed

These rules are ALWAYS ACTIVE for all subagent discovery and processing workflows, including SubagentsUtils.ts, SubagentsProcessor.ts, Copilot tool mapping, Claude agent propagation, and markdown file discovery operations.

### Rules

- **R-MSGQ-001** SHOULD: Collection structures SHOULD be passed to propagation functions (propagateSubagentsForClaude) to enable integration with external tools.
- **R-MSGQ-002** MUST: All subagent discovery operations use Set-based collections with .add() accumulation patterns to guarantee uniqueness.
- **R-MSGQ-003** MUST: Discovery functions (discoverSubagents, loadSubagentFile) accept collection references as parameters to enable accumulation across multiple invocations.
- **R-MSGQ-004** SHOULD: Use TypeScript Set<T> with appropriate type parameters for all subagent target collections.
- **R-MSGQ-005** MUST: No duplicate subagent targets appear in propagated collections to external tools.

### Verify

```bash
# Verify Set-based collection patterns in discovery operations
grep -r '\.add(' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts | grep -E '(mapped|targets)'

# Verify Set type declarations
grep -r 'Set<' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts

# Run collection and uniqueness tests
npm test -- --testPathPattern='Subagents(Utils|Processor)' --testNamePattern='collection|uniqueness'

# Verify no array-based collections bypass uniqueness guarantees
grep -r 'push(' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts | grep -v 'Set' || echo 'No unsafe array patterns found'
```

**Accept when:**
- All subagent discovery operations use Set-based collections with .add() accumulation patterns
- No duplicate subagent targets appear in propagated collections to external tools
- Tests verify uniqueness guarantees and proper accumulation across multiple discovery invocations
- Discovery functions accept collection references as parameters
- No array-based collections are used without explicit uniqueness validation

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests introducing subagent discovery code must pass these verification commands before merge. Violations of R-MSGQ-002, R-MSGQ-003, or R-MSGQ-005 block merge until corrected.
</enforcement>