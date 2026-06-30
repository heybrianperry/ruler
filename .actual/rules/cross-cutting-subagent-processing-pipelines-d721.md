# Adopt Message Queue Pattern for Subagent Target Collection: Subagent Processing Pipelines

These rules are ALWAYS ACTIVE for all subagent discovery and processing workflows, including SubagentsProcessor.ts, SubagentsUtils.ts, Copilot tool mapping, Claude agent propagation, and markdown file discovery operations.

### Rules

- **R-SUBAGENT-001** SHOULD: Subagent processing pipelines SHOULD separate discovery (SubagentsProcessor) from utility operations (SubagentsUtils) to maintain clear boundaries.
- **R-SUBAGENT-002** MUST: All subagent target collections MUST use TypeScript Set<T> with .add() accumulation patterns to guarantee uniqueness and prevent duplicate processing.
- **R-SUBAGENT-003** MUST: All discovery functions (discoverSubagents, loadSubagentFile) MUST accept collection references as parameters to enable incremental accumulation across multiple invocations.
- **R-SUBAGENT-004** SHOULD: Subagent discovery operations SHOULD document thread-safety requirements and implement synchronization mechanisms if parallel discovery is introduced.
- **R-SUBAGENT-005** MAY: Wrapper classes around Set MAY be implemented if additional metadata (discovery order, timestamps) needs to be tracked alongside targets.

### Verify

```bash
# Verify Set-based collection patterns in core subagent modules
grep -r '\.add(' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts | grep -E '(mapped|targets)'

# Verify Set type declarations
grep -r 'Set<' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts

# Run subagent-specific tests for collection and uniqueness behavior
npm test -- --testPathPattern='Subagents(Utils|Processor)' --testNamePattern='collection|uniqueness'
```

**Accept when:**
- All subagent discovery operations use Set-based collections with .add() accumulation patterns
- No duplicate subagent targets appear in propagated collections to external tools (Copilot, Claude)
- Tests verify uniqueness guarantees and proper accumulation across multiple discovery invocations
- Discovery and utility operations maintain clear separation of concerns
- Collection references are properly threaded through discovery function signatures

<enforcement>
Clause Code MUST NOT skip or defer verification. Pull requests introducing array-based collections without uniqueness guarantees must be revised. Discovery operations that bypass Set.add() patterns trigger code review feedback. Failed uniqueness tests block merge until collection patterns are corrected. Deviations from Set-based pattern require documented technical justification, demonstration of equivalent uniqueness guarantees, and architecture review approval.
</enforcement>