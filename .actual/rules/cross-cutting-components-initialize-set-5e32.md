# Use Set-Based Message Queue Pattern for Deduplication in Agent and Skill Processing: Components Initialize Set

These rules are ALWAYS ACTIVE for all agent and skill processing components that handle collections requiring deduplication, including agent configuration processing, skill discovery and propagation, and boundary coordination code that prevents duplicate operations.

### Rules

- **R-DEDUP-001** SHOULD: Components SHOULD initialize Set collections at the beginning of processing functions that aggregate items from multiple iterations or sources.

### Verify

```bash
# Check for Set.add() usage in agent and skill processing
grep -r '\.add(' src/agents/ src/core/ | grep -E '(seen|targets|keys|paths)\.add'

# Verify Set initialization patterns
grep -r 'new Set()' src/agents/ src/core/

# Run deduplication and uniqueness tests
npm test -- --grep 'deduplication|unique'
```

**Accept when:**
- Set.add() usage is found in FirebenderAgent and SkillsProcessor for collection building
- No manual array deduplication logic (filter + includes) exists in boundary coordination code
- Tests verify that duplicate items are not processed multiple times
- Set-to-array conversions use Array.from() or spread operator when passing to external APIs

<enforcement>
Claude Code MUST NOT skip or defer verification. All collection-building code in agent and skill processing boundaries MUST be reviewed for Set-based deduplication patterns. Code review feedback MUST request refactoring to Set-based approach when array-based deduplication is detected.
</enforcement>