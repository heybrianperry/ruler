# In-Memory Map Cache for Transient Entity Lookups: Components Not Rely Localized Memory Collection

These rules are ALWAYS ACTIVE for all transient entity lookups, configuration memoization, and in-memory collection usages across module execution lifecycles.

### Rules

- **R-MEM-001** MUST_NOT: Components MUST NOT rely on localized in-memory collection state for data requiring cross-process synchronization, durability, or distributed access.

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