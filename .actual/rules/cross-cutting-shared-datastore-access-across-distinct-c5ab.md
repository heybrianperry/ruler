# In-Memory Map Cache for Transient Entity Lookups: Shared Datastore Access Across Distinct Service

These rules are ALWAYS ACTIVE for all files involving transient key-value lookups, memoization of filesystem checks, and index configuration entities across modules.

### Rules

- **R-CA-001** SHOULD: Shared datastore access across distinct service boundaries SHOULD be mediated through a formalized datastore abstraction rather than raw collection instances.

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