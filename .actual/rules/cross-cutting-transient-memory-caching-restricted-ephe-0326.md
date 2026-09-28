# In-Memory Map Cache for Transient Entity Lookups: Transient Memory Caching Restricted Ephemeral Request

These rules are ALWAYS ACTIVE for all source code files implementing in-memory caching, entity lookups, configuration memoization, or temporary data storage using language built-in key-value collections.

### Rules

- **R-TRANSIENT-001** MUST: Transient in-memory caching MUST be restricted to ephemeral, request-scoped, or operation-scoped lifecycles when implemented using language built-in key-value collections.
- **R-TRANSIENT-002** MUST: Scope collection instances to the shortest viable lifespan, preferentially instantiating caches inside operation handlers rather than at module scope.
- **R-TRANSIENT-003** MUST: Verify that asynchronous operations populating cache entries handle rejection cleanly to prevent storing incomplete or corrupt state.
- **R-TRANSIENT-004** MUST: Invalidate or reinitialize cache mappings whenever source configuration files or paths are mutated to prevent stale cache persistence.

### Verify

```bash
find . -type f -exec grep -E "\.(set|get)\(" {} +
find . -maxdepth 2 -type f -exec grep -E '"test":|"scripts":' {} +
```

**Accept when:**
- All in-memory cache structures have defined lifecycle clearing or are scoped to transient operation execution.
- Project verification and test suites pass without memory leakage or stale cache regression errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by peer code review during pull request evaluation and automated static analysis and unit testing in continuous integration pipelines.
</enforcement>