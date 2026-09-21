# In-Memory Key-Value Caching Using Built-In Map Structures: Memory Cache Structures Not Used Shared

These rules are ALWAYS ACTIVE for all in-memory caching and transient indexing routines within single-run execution workflows and localized key-value lookups.

### Rules

- **R-CACHE-001** MUST_NOT: In-memory cache structures MUST NOT be used for shared cross-process persistence or durable datastore requirements without integrating an external datastore abstraction.
- **R-CACHE-002** MANDATORY: Discover and run the project's test suite to verify that in-memory cache operations preserve resolution correctness.
- **R-CACHE-003** MANDATORY: Execute the repository's static type checker and linter scripts to validate cache map key-value typings.

### Verify

```bash
# Discover and run the project's test suite
# (Command discovered from repository workspace)
# Execute static type checker and linter scripts
# (Command discovered from repository workspace)
```

**Accept when:**
- All unit and integration tests passing without memory leaks or state bleeding across test cases.
- Static analysis and type checking complete with zero errors regarding cache datastore interactions.

<enforcement>
Claude Code MUST NOT skip or defer verification. All changes introducing or modifying in-memory cache usage must be verified against these criteria.
</enforcement>