# Use Set-Based Message Queue Pattern for Deduplication in Agent and Skill Processing: Components That Accumulate

These rules are ALWAYS ACTIVE for all agent and skill processing components that handle collections requiring deduplication, including agent configuration processing, skill discovery and propagation, and any component that aggregates unique items from multiple sources.

### Rules

- **R-DEDUP-001** MUST: Components that accumulate items requiring uniqueness MUST use Set data structures (e.g., `seen.add(key)`, `targets.add(target)`) rather than arrays with manual deduplication.

### Verify

```bash
# Check for Set.add() usage in agent and skill processing
grep -r '\.add(' src/agents/ src/core/ | grep -E '(seen|targets|keys|paths)\.add'

# Verify Set initialization patterns
grep -r 'new Set()' src/agents/ src/core/

# Run deduplication-related tests
npm test -- --grep 'deduplication|unique'
```

**Accept when:**
- Set.add() usage is found in FirebenderAgent and SkillsProcessor for collection building
- No manual array deduplication logic (filter + includes) exists in boundary coordination code
- Tests verify that duplicate items are not processed multiple times
- Set-to-array conversions use `Array.from(targets)` or `[...targets]` when needed for API compatibility

<enforcement>
Claude Code MUST NOT skip or defer verification. All collection-building code in agent and skill processing components must be reviewed for Set-based deduplication patterns before acceptance.
</enforcement>