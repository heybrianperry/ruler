# Use Set-Based Message Queue Boundaries for Agent and Skill Processing: Components Extend Set

These rules are ALWAYS ACTIVE for all agent and skill processing components that handle configuration, discovery, and propagation workflows, including FirebenderAgent configuration management and SkillsProcessor discovery/propagation operations.

### Rules

- **R-QUEUE-001** MAY: Components MAY extend Set-based boundaries to other processing workflows where deduplication and idempotency are required.
- **R-QUEUE-002** MUST: Initialize Sets at the beginning of processing functions to establish clear boundary scope (e.g., `const seen = new Set()`, `const targets = new Set()`).
- **R-QUEUE-003** SHOULD: Use descriptive variable names that indicate the boundary purpose: 'seen' for tracking processed items, 'targets' for accumulating destinations, 'pending' for queued work.
- **R-QUEUE-004** SHOULD: When converting Sets to arrays for iteration or output, use `Array.from(set)` or `[...set]` spread syntax for clarity.
- **R-QUEUE-005** SHOULD: Log Set sizes at key processing milestones to track boundary growth and identify potential memory issues early.
- **R-QUEUE-006** MUST NOT: Use external message queue systems (RabbitMQ, Kafka, SQS), database-backed queue implementations, or persistent queue storage mechanisms for in-process agent and skill deduplication boundaries.
- **R-QUEUE-007** SHOULD: Convert Sets to sorted arrays before iteration when processing order matters, and document that Set-based boundaries do not guarantee order.
- **R-QUEUE-008** SHOULD: Implement Set size monitoring and periodic cleanup for long-running processes to mitigate memory exhaustion risk.

### Verify

```bash
# Check for Set.add() operations in agent and skill processing modules
grep -r 'seen\.add\|targets\.add' src/ --include='*.ts' | wc -l

# Count Set instantiations in Agent, Processor, and Skills components
grep -r 'new Set<' src/ --include='*.ts' | grep -E '(Agent|Processor|Skills)' | wc -l

# Check for deduplication-related tests
npm test -- --grep 'deduplication|Set.*boundary' 2>/dev/null || echo 'No specific tests found'

# Verify no manual array-based deduplication in Set-appropriate contexts
grep -r '\.filter.*includes\|\.includes.*filter' src/ --include='*.ts' | grep -E '(Agent|Processor|Skills)' | wc -l
```

**Accept when:**
- Grep commands identify at least 2 instances of `Set.add()` operations in agent or skill processing modules
- Code review confirms Set data structures are used for deduplication in public API contracts (FirebenderAgent, SkillsProcessor)
- No instances of manual array-based deduplication logic (filter, includes) exist in the same processing contexts where Sets are appropriate
- Set-based boundaries are documented in public API contracts as the deduplication strategy

<enforcement>
Claude Code MUST NOT skip or defer verification. All new agent and skill processing features MUST use Set-based boundaries for deduplication. Code review checklist MUST require Set-based boundaries. Automated grep-based verification in CI pipeline MUST check for Set.add() patterns in relevant modules. Architecture review MUST ensure deduplication strategy is documented for new public API contracts.
</enforcement>