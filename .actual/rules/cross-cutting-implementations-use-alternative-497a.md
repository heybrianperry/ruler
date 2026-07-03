# Adopt Message Queue Pattern for Subagent Target Collection: Implementations Use Alternative

These rules are ALWAYS ACTIVE for all subagent discovery and processing workflows, including SubagentsUtils.ts, SubagentsProcessor.ts, Copilot tool mapping, Claude agent propagation, and markdown file discovery operations.

### Rules

- **R-MSGQ-001** MAY: Implementations MAY use alternative collection patterns if they guarantee uniqueness and support asynchronous accumulation.

### Verify

```bash
# Verify Set-based collection patterns in subagent discovery
grep -r '\.add(' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts | grep -E '(mapped|targets)'

# Verify Set type declarations
grep -r 'Set<' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts

# Run collection and uniqueness tests
npm test -- --testPathPattern='Subagents(Utils|Processor)' --testNamePattern='collection|uniqueness'
```

**Accept when:**
- All subagent discovery operations use Set-based collections with .add() accumulation patterns
- No duplicate subagent targets appear in propagated collections to external tools
- Tests verify uniqueness guarantees and proper accumulation across multiple discovery invocations
- Alternative collection patterns (if used) demonstrate equivalent uniqueness guarantees through documented mechanisms

<enforcement>
Code review verification that new subagent discovery code uses Set-based collections or approved alternatives with uniqueness guarantees is mandatory. Unit tests validating uniqueness and accumulation behavior must pass. Integration tests confirming proper propagation to Copilot and Claude tools must pass. Pull requests introducing collection patterns without uniqueness guarantees must be revised. Claude Code MUST NOT skip or defer verification.
</enforcement>