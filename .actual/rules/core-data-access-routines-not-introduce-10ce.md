# FileSystemUtils In-Memory Range Access Pattern: Data Access Routines Not Introduce External

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-FSU-001** MUST_NOT: Data access routines MUST_NOT introduce external indexing abstractions or caching layers for ephemeral, single-invocation utility range lookups.

### Verify

```bash
# Discover and execute the repository test execution script covering utility data access routines
# Discover and run static analysis linters to confirm adherence to architectural data access boundaries
```

**Accept when:**
- Automated unit test suites for utility range resolution execute without failure.
- Static verification confirms data access logic does not introduce unapproved indexing dependencies into utility modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>