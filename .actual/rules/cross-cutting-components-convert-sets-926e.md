# Use Set-Based Message Queue Pattern for Deduplication in Agent and Skill Processing: Components Convert Sets

These rules are ALWAYS ACTIVE for all agent and skill processing components that handle collections requiring deduplication, including agent configuration processing, skill discovery and propagation, and boundary coordination code that prevents duplicate operations.

### Rules

- **R-DEDUP-001** MAY: Components MAY convert Sets to arrays when interfacing with APIs that require array parameters, using Array.from() or spread syntax.

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
- Set-to-array conversions use Array.from() or spread syntax at API boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review checks for Set usage in new collection-building code, unit tests verify deduplication behavior, and static analysis tools flag array-based deduplication patterns as violations requiring refactoring to Set-based approach.
</enforcement>