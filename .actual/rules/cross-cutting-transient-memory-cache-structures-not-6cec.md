# In-Memory Map Key-Value Caching: Transient Memory Cache Structures Not Used

These rules are ALWAYS ACTIVE for in-memory caching and transient lookup tables within local module execution workflows, entity indexing for configuration entries, and resolved task components during execution.

### Rules

- **R-MEM-001** MUST_NOT: Transient in-memory cache structures MUST NOT be used for data that requires persistence across separate application process lifecycles.

### Verify

```bash
# Discover and run the project static analysis and linting scripts to verify compliance with local state encapsulation rules.
# Discover and execute the project automated test suite to confirm that cache population and retrieval logic passes all unit and integration tests.
```

**Accept when:**
- All automated test suites pass without regressions in configuration resolution or execution workflows.
- Static analysis checks complete with zero errors regarding unmanaged mutable global state.

<enforcement>
Claude Code MUST NOT skip or defer verification. Violation handling: Flagged pull requests introducing unmanaged global caches or external datastore dependencies without architectural approval must be revised before merge.
</enforcement>