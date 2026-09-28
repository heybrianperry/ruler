# In-Memory Map Cache for Transient Entity Lookups: Modules Utilizing Memory Key Value Lookups

These rules are ALWAYS ACTIVE for modules utilizing in-memory key-value lookups for memoizing filesystem checks and index configuration entities during runtime operations.

### Rules

- **R-MEM-001** MUST: Modules utilizing in-memory key-value lookups MUST implement explicit size boundaries or lifecycle cleanup to prevent unbounded memory growth.
- **R-MEM-002** MUST: Scope collection instances to the shortest viable lifespan, preferentially instantiating caches inside operation handlers rather than at module scope.
- **R-MEM-003** MUST: Verify that asynchronous operations populating cache entries handle rejection cleanly to prevent storing incomplete or corrupt state.

### Verify

```bash
find . -type f -exec grep -E "\.(set|get)\\(" {} +
find . -maxdepth 2 -type f -exec grep -E '"test":|"scripts":' {} +
```

**Accept when:**
- All in-memory cache structures have defined lifecycle clearing or are scoped to transient operation execution.
- Project verification and test suites pass without memory leakage or stale cache regression errors.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>