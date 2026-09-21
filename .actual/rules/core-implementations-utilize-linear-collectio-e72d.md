# FileSystemUtils In-Memory Range Access Pattern: Implementations Utilize Linear Collection Scanning Exclusively

These rules are ALWAYS ACTIVE for internal utility modules managing file system pattern matching, boundary identification, and in-memory range segment queries.

### Rules

- **R-FSU-001** SHOULD: Implementations SHOULD utilize linear collection scanning exclusively for bounded collections where linear traversal overhead is negligible.
- **R-FSU-002** MANDATORY: Preserve immutability of range collections during iteration to prevent predicate evaluation inconsistencies.
- **R-FSU-003** MANDATORY: Profile collection sizes at runtime to verify that linear scanning remains within acceptable latency budgets for utility routines.

### Verify

```bash
# Discover the repository test execution script and run automated test suites covering utility data access routines
# Discover and run static analysis linters to confirm adherence to architectural data access boundaries
```

**Accept when:**
- Automated unit test suites for utility range resolution execute without failure.
- Static verification confirms data access logic does not introduce unapproved indexing dependencies into utility modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated continuous integration pipeline test execution and peer code review.
</enforcement>