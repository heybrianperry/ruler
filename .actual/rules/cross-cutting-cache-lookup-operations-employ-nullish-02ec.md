# In-Memory Key-Value Caching Using Built-In Map Structures: Cache Lookup Operations Employ Nullish Coalescing

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-CACHE-001** SHOULD: Cache lookup operations SHOULD employ nullish coalescing or explicit existence checks when retrieving keyed definitions to provide deterministic fallback handling for missing cache entries.

### Verify

```bash
# Discover and run the project's test suite to verify that in-memory cache operations preserve resolution correctness
# Execute the repository's static type checker and linter scripts to validate cache map key-value typings
```

**Accept when:**
- All unit and integration tests passing without memory leaks or state bleeding across test cases.
- Static analysis and type checking complete with zero errors regarding cache datastore interactions.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>