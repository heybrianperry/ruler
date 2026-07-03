# Standardize Array.find() for Safe Data Access in Test Assertions: Test Code Not

These rules are ALWAYS ACTIVE for all test files accessing parsed configuration data structures, including TOML-parsed MCP server lists and hierarchical ruler configurations.

### Rules

- **R-42-001** MUST NOT: Test code MUST NOT use direct array indexing (e.g., array[0]) to access parsed configuration objects without prior length validation.
- **R-42-002** MUST: Test assertions accessing parsed configuration arrays MUST use Array.find() with explicit predicates to locate specific configuration objects by name or path properties.
- **R-42-003** MUST: Test assertions MUST include explicit undefined checks before accessing properties on find() results (e.g., expect(found).toBeDefined(); expect(found?.property).toBe(expectedValue)).
- **R-42-004** SHOULD: Use descriptive predicate variable names that clarify the search intent (e.g., (s: any) => s.name === 'expected_server' rather than (s) => s.name === 'expected_server').
- **R-42-005** SHOULD: For tests validating multiple configuration objects, extract find() calls to test setup (beforeEach) with clear variable names to reduce repetition.
- **R-42-006** MAY: Create custom test utility functions (e.g., findConfigByName) that wrap Array.find() when multiple test files show repetitive Array.find() patterns with identical predicate logic.

Exceptions:
- **EXC-001**: Test explicitly validates array indexing behavior or boundary conditions.
- **EXC-002**: Test uses array destructuring with default values that provide safe fallbacks.

### Verify

```bash
# Flag direct indexing without length checks on parsed configuration objects
grep -r 'parsed.*\[0\]' tests/ --include='*.test.ts' | grep -v 'length'

# Count Array.find() usage in tests
grep -r '\.find(' tests/ --include='*.test.ts' | wc -l

# Verify reference tests pass
npm test -- --testPathPattern='MistralVibeAgent|apply-engine'
```

**Accept when:**
- All test files accessing parsed configuration arrays use Array.find() with explicit predicates.
- Test assertions include explicit undefined checks before accessing properties on find() results.
- Grep search for direct array indexing on parsed configuration objects returns zero results in test files.
- At least one negative test case per configuration type verifies find() returns undefined for non-existent entries.
- Reference tests (MistralVibeAgent.test.ts and apply-engine.test.ts) pass with all assertions intact.

<enforcement>
Clause Code MUST NOT skip or defer verification. All test files accessing parsed configuration data MUST conform to R-42-001 through R-42-005. Violations flagged by grep checks or code review MUST be remediated before merge. Exceptions MUST be documented with EXC-ID reference and approved by code reviewer.
</enforcement>