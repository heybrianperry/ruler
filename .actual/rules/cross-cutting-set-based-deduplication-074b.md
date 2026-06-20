# Use Set-Based Message Queue Pattern for Deduplication in Agent and Skill Processing: Set Based Deduplication

These rules are ALWAYS ACTIVE for all agent and skill processing components that handle collections requiring deduplication, including agent configuration processing, skill discovery and propagation, and boundary coordination code that prevents duplicate operations.

### Rules

- **R-DEDUP-001** SHOULD: Set-based deduplication SHOULD be preferred over array filtering or includes() checks for performance-sensitive collection building.
- **R-DEDUP-002** MUST: Initialize Sets at the beginning of functions that aggregate items using `const seen = new Set();` or `const targets = new Set();`.
- **R-DEDUP-003** MUST: Use Set.add() in loops or callbacks where items are accumulated: `seen.add(key);` or `targets.add(target);`.
- **R-DEDUP-004** MUST: Convert Sets to arrays when needed for API compatibility using `Array.from(targets)` or `[...targets]`.
- **R-DEDUP-005** SHOULD: Use Set.has() for efficient membership testing before conditional operations.
- **R-DEDUP-006** MUST NOT: Use manual array deduplication logic (filter + includes) in boundary coordination code.

### Verify

```bash
# Find Set.add() usage in agent and skill processing
grep -r '\.add(' src/agents/ src/core/ | grep -E '(seen|targets|keys|paths)\.add'

# Find all Set instantiations
grep -r 'new Set()' src/agents/ src/core/

# Run deduplication and uniqueness tests
npm test -- --grep 'deduplication|unique'

# Check for array-based deduplication patterns (should be minimal/absent)
grep -r 'filter.*includes' src/agents/ src/core/ | grep -v node_modules
```

**Accept when:**
- Set.add() usage is found in FirebenderAgent and SkillsProcessor for collection building
- No manual array deduplication logic (filter + includes) exists in boundary coordination code
- Tests verify that duplicate items are not processed multiple times
- Set-to-array conversions are present at API boundaries where needed
- Set.has() is used for membership testing in performance-sensitive paths

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for agent and skill processing code paths. Code review and static analysis MUST flag violations of R-DEDUP-001 through R-DEDUP-006.
</enforcement>