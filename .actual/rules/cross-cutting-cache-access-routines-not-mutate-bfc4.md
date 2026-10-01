# In-Memory Map Key-Value Caching: Cache Access Routines Not Mutate Stored

These rules are ALWAYS ACTIVE for in-memory caching and transient lookup tables within local module execution workflows, and entity indexing for configuration entries and resolved task components during execution.

### Rules

- **R-CACHE-001** SHOULD_NOT: Cache access routines SHOULD NOT mutate stored entity state directly upon retrieval without explicit defensive copying when immutability is required.

### Verify

```bash
# Discover and run the project static analysis and linting scripts
# Discover and execute the project automated test suite
```

**Accept when:**
- All automated test suites pass without regressions in configuration resolution or execution workflows.
- Static analysis checks complete with zero errors regarding unmanaged mutable global state.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated test validation and peer code review.
</enforcement>