# Use Set-Based Message Queue Boundaries for Agent and Skill Processing: Components Use Descriptive

These rules are ALWAYS ACTIVE for all agent and skill processing components that handle configuration, discovery, and propagation workflows, including FirebenderAgent configuration management and SkillsProcessor discovery/propagation operations.

### Rules

- **R-QUEUE-001** SHOULD: Components SHOULD use descriptive Set variable names (seen, targets) that clearly indicate the boundary purpose and scope.

### Verify

```bash
# Check for Set.add() operations in agent and skill processing modules
grep -r 'seen\.add\|targets\.add' src/ --include='*.ts' | wc -l

# Count Set instantiations in Agent/Processor/Skills contexts
grep -r 'new Set<' src/ --include='*.ts' | grep -E '(Agent|Processor|Skills)' | wc -l

# Check for deduplication or Set boundary tests
npm test -- --grep 'deduplication|Set.*boundary' 2>/dev/null || echo 'No specific tests found'
```

**Accept when:**
- Grep commands identify at least 2 instances of Set.add() operations in agent or skill processing modules
- Code review confirms Set data structures are used for deduplication in public API contracts
- No instances of manual array-based deduplication logic (filter, includes) exist in the same processing contexts where Sets are appropriate

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent and skill processing components introducing collection-based deduplication MUST use Set-based boundaries with descriptive variable names and MUST pass the verify commands before acceptance.
</enforcement>