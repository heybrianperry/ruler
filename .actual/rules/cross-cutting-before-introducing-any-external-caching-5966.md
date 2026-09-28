# In-Memory Map Cache for Transient Entity Lookups: Before Introducing Any External Caching Datastore

These rules are ALWAYS ACTIVE for all transient key-value lookups, memoization of filesystem checks, and index configuration entities during runtime operations.

### Rules

- **R-CACHE-001** MUST: Before introducing any external caching or datastore library dependency, the consumer MUST inspect the project manifest and lock artifact to verify dependency governance compliance.

### Verify

```bash
find . -type f -exec grep -E "\.(set|get)\(" {} +
find . -maxdepth 2 -type f -exec grep -E '"test":|"scripts":' {} +
```

**Accept when:**
- All in-memory cache structures have defined lifecycle clearing or are scoped to transient operation execution.
- Project verification and test suites pass without memory leakage or stale cache regression errors.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>