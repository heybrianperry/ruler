# FileSystemUtils In-Memory Range Access Pattern: Modules Performing Memory Range Lookup Across

These rules are ALWAYS ACTIVE for all code performing in-memory range lookups across internal FileSystemUtils structures.

### Rules

- **R-FSU-001** MUST: Modules performing in-memory range lookup across internal FileSystemUtils structures MUST evaluate the cardinality of collections before adopting linear predicate searches.

### Verify

```bash
# Discover the repository test execution script and run automated test suites covering utility data access routines.
# Discover and run static analysis linters to confirm adherence to architectural data access boundaries.
```

**Accept when:**
- Automated unit test suites for utility range resolution execute without failure.
- Static verification confirms data access logic does not introduce unapproved indexing dependencies into utility modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>