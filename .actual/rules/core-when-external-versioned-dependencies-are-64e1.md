# FileSystemUtils In-Memory Range Access Pattern: When External Versioned Dependencies Are Introduced

These rules are ALWAYS ACTIVE for all code and configuration interacting with in-memory range access patterns and external versioned dependencies within utility modules.

### Rules

- **R-FSU-001** MUST: When external versioned dependencies are introduced for data access or storage, consumers MUST inspect the repository lock artifact and verify the exact resolved version against authorized API specifications prior to adoption.
- **R-FSU-002** MANDATORY (DISCOVERY POLICY): Omit all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.
- **R-FSU-003** MANDATORY (LOCK-VERSION GROUNDING): Before writing code that uses a versioned library, execute in order: (1) Find the dependency manifest in the repo. (2) Identify the build tool. (3) Inspect the repository lock or resolution artifact to determine the exact resolved version. (4) Look up official documentation/changelog/public API reference for that exact version (no training-data recall). (5) Confirm every API, class, or function used exists in that exact version's documentation. (6) For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- **R-FSU-004** MUST: Profile collection sizes at runtime to verify that linear scanning remains within acceptable latency budgets for utility routines.
- **R-FSU-005** MUST: Preserve immutability of range collections during iteration to prevent predicate evaluation inconsistencies.

### Verify

```bash
# Discover and run the repository test execution script covering utility data access routines
# Discover and run static analysis linters to confirm adherence to architectural data access boundaries
```

**Accept when:**
- Automated unit test suites for utility range resolution execute without failure.
- Static verification confirms data access logic does not introduce unapproved indexing dependencies into utility modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated test execution and peer code review for pull requests modifying utility data access patterns.
</enforcement>