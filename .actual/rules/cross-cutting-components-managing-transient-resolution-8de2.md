# In-Memory Key-Value Caching Using Built-In Map Structures: Components Managing Transient Resolution State Utilize

These rules are ALWAYS ACTIVE for components managing transient resolution state, configuration traversal, and protocol server propagation routines.

### Rules

- **R-CACHE-001** MUST: Components managing transient resolution state MUST utilize localized in-memory key-value map structures with set and get operations to isolate runtime cache entries within the active process execution scope.

### Verify

```bash
# Discover and run the project's test suite to verify that in-memory cache operations preserve resolution correctness.
# Execute the repository's static type checker and linter scripts to validate cache map key-value typings.
```

**Accept when:**
- All unit and integration tests passing without memory leaks or state bleeding across test cases.
- Static analysis and type checking complete with zero errors regarding cache datastore interactions.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated continuous integration test execution checking for test isolation and deterministic behavior, and peer code review verifying that cache lifecycles remain bounded to appropriate execution contexts.
</enforcement>