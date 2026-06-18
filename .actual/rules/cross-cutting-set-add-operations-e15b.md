# Use Set-Based Message Queue Boundaries for Agent and Skill Processing: Set Add Operations

These rules are ALWAYS ACTIVE for all agent and skill processing components that handle configuration, discovery, and propagation workflows, including FirebenderAgent configuration management and SkillsProcessor discovery/propagation operations.

### Rules

- **R-SET-001** MUST: Set.add() operations MUST be used to accumulate items during concurrent operations to ensure automatic deduplication in agent configuration management (applyRulerConfig, loadExistingConfig, saveConfig, handleMcpConfiguration) and skill processing (discoverSkills, getSkillsGitignorePaths, propagateSkills, propagateSkillsForClaude, propagateSkillsForCodex).

- **R-SET-002** MUST: Initialize Sets at the beginning of processing functions with descriptive variable names that indicate boundary purpose (e.g., 'seen' for tracking processed items, 'targets' for accumulating destinations, 'pending' for queued work).

- **R-SET-003** SHOULD: Convert Sets to arrays using Array.from(set) or [...set] spread syntax when iteration order or output clarity is required.

- **R-SET-004** SHOULD: Log Set sizes at key processing milestones to track boundary growth and identify potential memory issues early in long-running processes.

- **R-SET-005** SHOULD: Convert Sets to sorted arrays before iteration when processing order matters, and document that Set-based boundaries do not guarantee order.

- **R-SET-006** MAY: Transition to Map data structure with boolean values or additional metadata when tracking timestamps, status, or retry counts alongside deduplication boundaries.

### Verify

```bash
# Count Set.add() operations in agent and skill processing modules
grep -r 'seen\.add\|targets\.add' src/ --include='*.ts' | wc -l

# Count Set instantiations in Agent/Processor/Skills contexts
grep -r 'new Set<' src/ --include='*.ts' | grep -E '(Agent|Processor|Skills)' | wc -l

# Check for deduplication or Set boundary tests
npm test -- --grep 'deduplication|Set.*boundary' 2>/dev/null || echo 'No specific tests found'

# Verify no manual array-based deduplication in Set-appropriate contexts
grep -r '\.filter.*includes\|\.includes.*filter' src/ --include='*.ts' | grep -E '(Agent|Processor|Skills)' | wc -l
```

**Accept when:**
- Grep commands identify at least 2 instances of Set.add() operations in agent or skill processing modules
- Code review confirms Set data structures are used for deduplication in public API contracts
- No instances of manual array-based deduplication logic (filter, includes) exist in the same processing contexts where Sets are appropriate
- Set-based boundaries are documented in public API contracts for agent and skill processing

<enforcement>
Claude Code MUST NOT skip or defer verification. All Set-based message queue boundaries in agent and skill processing must be verified against the grep commands and acceptance criteria before code review approval.
</enforcement>