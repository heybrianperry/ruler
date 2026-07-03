# Use Set-Based Message Queue Boundaries for Agent and Skill Processing: Public Contracts Agent

These rules are ALWAYS ACTIVE for all agent and skill processing components that handle configuration, discovery, and propagation workflows, including FirebenderAgent configuration management and SkillsProcessor discovery and propagation operations.

### Rules

- **R-SETQ-001** MUST: Public API contracts for agent configuration (FirebenderAgent) and skill processing (discoverSkills, propagateSkills, propagateSkillsForClaude, propagateSkillsForCodex) MUST maintain Set-based boundaries throughout their execution lifecycle.
- **R-SETQ-002** MUST: Initialize Sets at the beginning of processing functions (e.g., `const seen = new Set()`, `const targets = new Set()`) to establish clear boundary scope.
- **R-SETQ-003** SHOULD: Use descriptive variable names that indicate the boundary purpose: 'seen' for tracking processed items, 'targets' for accumulating destinations, 'pending' for queued work.
- **R-SETQ-004** SHOULD: When converting Sets to arrays for iteration or output, use `Array.from(set)` or `[...set]` spread syntax for clarity.
- **R-SETQ-005** SHOULD: Log Set sizes at key processing milestones to track boundary growth and identify potential memory issues early.
- **R-SETQ-006** SHOULD: Convert Sets to sorted arrays before iteration when processing order matters, and document that Set-based boundaries do not guarantee order.
- **R-SETQ-007** MAY: Implement Set size monitoring and periodic cleanup for long-running processes; consider adding maximum size limits with overflow handling for production deployments.

### Verify

```bash
# Check for Set.add() operations in agent and skill processing modules
grep -r 'seen\.add\|targets\.add' src/ --include='*.ts' | wc -l

# Check for Set instantiation in Agent or Processor contexts
grep -r 'new Set<' src/ --include='*.ts' | grep -E '(Agent|Processor|Skills)' | wc -l

# Check for deduplication or Set boundary tests
npm test -- --grep 'deduplication|Set.*boundary' 2>/dev/null || echo 'No specific tests found'
```

**Accept when:**
- Grep commands identify at least 2 instances of Set.add() operations in agent or skill processing modules
- Code review confirms Set data structures are used for deduplication in public API contracts
- No instances of manual array-based deduplication logic (filter, includes) exist in the same processing contexts where Sets are appropriate

<enforcement>
Claude Code MUST NOT skip or defer verification. All new agent and skill processing features MUST be reviewed against these rules. Violations require refactoring to Set-based boundaries or documented exception approval.
</enforcement>