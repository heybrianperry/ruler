# In-Memory Map Key-Value Caching: Memory Cache Entries Use Unique Deterministic

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-MEM-001** SHOULD: In-memory cache entries SHOULD use unique, deterministic string identifiers derived directly from configuration paths or entity names as map keys.

### Verify

```bash
# Discover and run static analysis and linting scripts
# Discover and execute project automated test suites
```

**Accept when:**
- All automated test suites pass without regressions in configuration resolution or execution workflows.
- Static analysis checks complete with zero errors regarding unmanaged mutable global state.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>