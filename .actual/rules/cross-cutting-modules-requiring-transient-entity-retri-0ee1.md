# In-Memory Map Key-Value Caching: Modules Requiring Transient Entity Retrieval Isolate

These rules are ALWAYS ACTIVE for modules requiring transient entity retrieval and local in-memory key-value caching.

### Rules

- **R-CACHE-001** MUST: Modules requiring transient entity retrieval MUST isolate key-value storage to process-local in-memory map structures rather than introducing unmanaged global state or external cache daemons.

### Verify

```bash
# Discover and run the project static analysis and linting scripts to verify compliance with local state encapsulation rules.
# Discover and execute the project automated test suite to confirm that cache population and retrieval logic passes all unit and integration tests.
```

**Accept when:**
- All automated test suites pass without regressions in configuration resolution or execution workflows.
- Static analysis checks complete with zero errors regarding unmanaged mutable global state.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated test validation and peer code review.
</enforcement>