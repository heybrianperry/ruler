# In-Memory Key-Value Caching Using Built-In Map Structures: Memory Cache Instances Not Maintain Unbounded

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-CACHE-001** SHOULD_NOT: In-memory cache instances SHOULD_NOT maintain unbounded references across long-lived daemon processes without defining an explicit eviction or lifecycle reset boundary.
- **R-CACHE-002** MANDATORY: Discover the project's dependency manifest, build tool, and lock/resolution artifact to ground version-sensitive behavior before writing cache-related code using external libraries.
- **R-CACHE-003** MANDATORY: Instantiate cache map instances at the top of the evaluation workflow and pass them down or encapsulate them within the resolution coordinator to ensure proper garbage collection after execution.
- **R-CACHE-004** MANDATORY: Use normalized absolute string paths or canonical URLs as lookup keys to avoid cache misses caused by equivalent but differently formatted keys.

### Verify

```bash
# Discover and run the project's test suite to verify that in-memory cache operations preserve resolution correctness.
# Execute the repository's static type checker and linter scripts to validate cache map key-value typings.
```

**Accept when:**
- All unit and integration tests passing without memory leaks or state bleeding across test cases.
- Static analysis and type checking complete with zero errors regarding cache datastore interactions.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>