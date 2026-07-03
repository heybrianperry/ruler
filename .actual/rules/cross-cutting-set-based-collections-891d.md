# Use Set-Based Message Queue Pattern for Deduplication in Agent and Skill Processing: Set Based Collections

These rules are ALWAYS ACTIVE for all agent and skill processing components that handle collections requiring deduplication, including agent configuration processing, skill discovery and propagation, and any component that aggregates unique items from multiple sources at boundary coordination points.

### Rules

- **R-SETDEDUP-001** MUST: Set-based collections MUST be used at boundaries where multiple sources contribute to a single collection (e.g., configuration keys, skill targets, file paths).
- **R-SETDEDUP-002** MUST: Initialize Sets at the beginning of functions that aggregate items using `const seen = new Set();` or `const targets = new Set();` pattern.
- **R-SETDEDUP-003** MUST: Use `Set.add()` in loops or callbacks where items are accumulated rather than array push operations.
- **R-SETDEDUP-004** SHOULD: Convert Sets to arrays when needed for API compatibility using `Array.from(targets)` or `[...targets]` syntax.
- **R-SETDEDUP-005** SHOULD: Use `Set.has()` for efficient membership testing before conditional operations instead of array includes().
- **R-SETDEDUP-006** MUST NOT: Use manual array deduplication logic (filter + includes) in boundary coordination code where Set-based approach is applicable.

### Verify

```bash
# Find Set.add() usage in agent and skill processing files
grep -r '\.add(' src/agents/ src/core/ | grep -E '(seen|targets|keys|paths)\.add'

# Find all Set instantiations in scope
grep -r 'new Set()' src/agents/ src/core/

# Run deduplication-related tests
npm test -- --grep 'deduplication|unique'

# Check for manual array deduplication patterns (should return minimal results)
grep -r 'filter.*includes\|includes.*filter' src/agents/ src/core/ | grep -v node_modules
```

**Accept when:**
- Set.add() usage is found in FirebenderAgent and SkillsProcessor for collection building at boundaries
- No manual array deduplication logic (filter + includes) exists in boundary coordination code
- Tests verify that duplicate items are not processed multiple times
- Set-to-array conversions are present where external APIs require array parameters
- Code review confirms Set-based approach is used for all multi-source collection aggregation

<enforcement>
Claude Code MUST NOT skip or defer verification of Set-based collection usage in agent and skill processing boundary code. All collection-building code at boundaries MUST be reviewed for Set compliance before acceptance.
</enforcement>