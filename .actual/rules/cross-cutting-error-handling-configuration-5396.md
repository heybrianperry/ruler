# Use Set-Based Message Queue Boundaries for Agent and Skill Processing: Error Handling Configuration

These rules are ALWAYS ACTIVE for all agent and skill processing components that handle configuration, discovery, and propagation workflows, including FirebenderAgent configuration management and SkillsProcessor discovery/propagation operations.

### Rules

- **R-QUEUE-001** SHOULD: Error handling for configuration parsing (JSON.parse) SHOULD log failures using console.warn to maintain visibility into boundary violations without blocking processing.

### Verify

```bash
# Verify Set-based boundary patterns in agent and skill processing modules
grep -r 'seen\.add\|targets\.add' src/ --include='*.ts' | wc -l

# Count Set instantiations in Agent and Processor contexts
grep -r 'new Set<' src/ --include='*.ts' | grep -E '(Agent|Processor|Skills)' | wc -l

# Check for deduplication-related tests
npm test -- --grep 'deduplication|Set.*boundary' 2>/dev/null || echo 'No specific tests found'
```

**Accept when:**
- Grep commands identify at least 2 instances of Set.add() operations in agent or skill processing modules
- Code review confirms Set data structures are used for deduplication in public API contracts
- No instances of manual array-based deduplication logic (filter, includes) exist in the same processing contexts where Sets are appropriate
- console.warn is used for JSON.parse error handling in configuration parsing paths

<enforcement>
Claude Code MUST NOT skip or defer verification of Set-based boundary patterns and error handling configuration in agent and skill processing components.
</enforcement>