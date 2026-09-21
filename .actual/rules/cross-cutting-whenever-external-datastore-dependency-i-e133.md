# In-Memory Key-Value Caching Using Built-In Map Structures: Whenever External Datastore Dependency Introduced Replace

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-CACH-001** MUST: Whenever an external datastore dependency is introduced to replace or augment in-memory caching, the consumer MUST discover the project dependency manifest and resolve the exact locked version from the lock artifact before implementation.
- **R-CACH-002** MUST: Instantiate cache map instances at the top of the evaluation workflow and pass them down or encapsulate them within the resolution coordinator to ensure proper garbage collection after execution.
- **R-CACH-003** MUST: Use normalized absolute string paths or canonical URLs as lookup keys to avoid cache misses caused by equivalent but differently formatted keys.
- **R-CACH-004** MUST: Scope cache instances strictly to the lifetime of single execution requests or commands to prevent unbounded memory growth.
- **R-CACH-005** MUST: Clear or recreate cache collections whenever a mutation or file system modification is triggered to prevent stale data returned from extended process lifetimes.

### Verify

```bash
# Discover and run the project's test suite to verify that in-memory cache operations preserve resolution correctness.
# Execute the repository's static type checker and linter scripts to validate cache map key-value typings.
```

**Accept when:**
- All unit and integration tests passing without memory leaks or state bleeding across test cases.
- Static analysis and type checking complete with zero errors regarding cache datastore interactions.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory.
</enforcement>