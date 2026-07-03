# Use Set-Based Message Queue Boundaries for Agent and Skill Processing: Agent Skill Processing

These rules are ALWAYS ACTIVE for all agent and skill processing components that handle configuration, discovery, and propagation workflows, including FirebenderAgent configuration management and SkillsProcessor discovery/propagation operations.

### Rules

- **R-SETQ-001** MUST: Agent and skill processing components MUST use Set data structures to establish message queue boundaries for deduplication of configuration keys and processing targets.
- **R-SETQ-002** MUST: Initialize Sets at the beginning of processing functions with descriptive variable names indicating boundary purpose (e.g., `const seen = new Set()`, `const targets = new Set()`).
- **R-SETQ-003** SHOULD: Use descriptive variable names that indicate the boundary purpose: 'seen' for tracking processed items, 'targets' for accumulating destinations, 'pending' for queued work.
- **R-SETQ-004** SHOULD: When converting Sets to arrays for iteration or output, use `Array.from(set)` or `[...set]` spread syntax for clarity.
- **R-SETQ-005** SHOULD: Log Set sizes at key processing milestones to track boundary growth and identify potential memory issues early.
- **R-SETQ-006** SHOULD: Convert Sets to sorted arrays before iteration when processing order matters, and document that Set-based boundaries do not guarantee order.
- **R-SETQ-007** MAY: Use Map data structure with additional metadata (timestamps, status, retry counts) when tracking processed items requires more than simple deduplication.

### Verify

```bash
# Count Set.add() operations in agent and skill processing modules
grep -r 'seen\.add\|targets\.add' src/ --include='*.ts' | wc -l

# Count Set instantiations in Agent/Processor/Skills contexts
grep -r 'new Set<' src/ --include='*.ts' | grep -E '(Agent|Processor|Skills)' | wc -l

# Check for deduplication-related tests
npm test -- --grep 'deduplication|Set.*boundary' 2>/dev/null || echo 'No specific tests found'

# Verify no manual array-based deduplication in agent/skill processing
grep -r '\.filter\|includes' src/ --include='*.ts' | grep -E '(FirebenderAgent|SkillsProcessor)' | wc -l
```

**Accept when:**
- Grep commands identify at least 2 instances of Set.add() operations in agent or skill processing modules
- Code review confirms Set data structures are used for deduplication in public API contracts
- No instances of manual array-based deduplication logic (filter, includes) exist in the same processing contexts where Sets are appropriate
- Set-based boundaries are documented in public API contracts for agent and skill processing functions

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent and skill processing components introducing collection deduplication MUST be verified against these rules before acceptance.
</enforcement>