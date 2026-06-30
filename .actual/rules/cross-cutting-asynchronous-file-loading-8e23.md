# Adopt Message Queue Pattern for Subagent Target Collection: Asynchronous File Loading

These rules are ALWAYS ACTIVE for all subagent discovery and processing workflows, including file loading operations in SubagentsUtils.ts, SubagentsProcessor.ts, Copilot tool mapping, Claude agent propagation, and markdown file discovery with frontmatter parsing.

### Rules

- **R-MSGQ-001** MUST: Asynchronous file loading operations (loadSubagentFile, discoverSubagents) MUST populate shared collection structures for downstream processing.
- **R-MSGQ-002** MUST: All subagent target collections MUST use TypeScript Set<T> with appropriate type parameters to guarantee uniqueness and enable O(1) duplicate detection.
- **R-MSGQ-003** MUST: All discovery functions (discoverSubagents, loadSubagentFile) MUST accept collection references as parameters to enable accumulation across multiple invocations.
- **R-MSGQ-004** MUST: Subagent target collections MUST support add() operations and uniqueness guarantees before propagation to external tools (Copilot, Claude).
- **R-MSGQ-005** SHOULD: Maintain separate logging of discovery order or parallel ordered collections for diagnostic purposes when debugging subagent loading failures.
- **R-MSGQ-006** SHOULD: Document thread-safety requirements and implement mutex locks or atomic operations if parallel discovery operations are introduced.
- **R-MSGQ-007** MAY: Implement a wrapper class around Set if additional metadata (discovery order, timestamps) needs to be tracked alongside targets.

### Verify

```bash
# Verify Set-based collection patterns in subagent utilities and processor
grep -r '\.add(' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts | grep -E '(mapped|targets)'

# Verify Set type declarations
grep -r 'Set<' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts

# Run subagent collection and uniqueness tests
npm test -- --testPathPattern='Subagents(Utils|Processor)' --testNamePattern='collection|uniqueness'
```

**Accept when:**
- All subagent discovery operations use Set-based collections with .add() accumulation patterns
- No duplicate subagent targets appear in propagated collections to external tools (Copilot, Claude)
- Tests verify uniqueness guarantees and proper accumulation across multiple discovery invocations
- grep commands confirm Set<> type usage and .add() patterns in target collection code
- Unit tests pass for collection uniqueness and accumulation behavior
- Integration tests confirm proper propagation to Copilot and Claude tools without duplicates

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-MSGQ rules are mandatory for subagent discovery code. Pull requests introducing array-based collections without uniqueness guarantees must be revised. Discovery operations that bypass Set.add() patterns trigger code review feedback. Failed uniqueness tests block merge until collection patterns are corrected.
</enforcement>