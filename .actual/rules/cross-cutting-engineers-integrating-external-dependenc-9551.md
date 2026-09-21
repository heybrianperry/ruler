# IAgent Core Agent Interface Module Adoption: Engineers Integrating External Dependencies Updating Runtime

These rules are ALWAYS ACTIVE for orchestration modules executing agent tasks, managing agent lifecycles, and rollback/revert routines coordinating agent-specific reversal operations across directory configurations.

### Rules

- **R-DEP-001** MUST: Engineers integrating external dependencies or updating runtime libraries MUST discover the project dependency manifest and lock file to resolve and pin the exact dependency versions.
- **R-DEP-002** MANDATORY: Before writing code that uses a versioned library, execute in order: (1) Find the dependency manifest in the repo declaring ranges, (2) Identify the build tool from the manifest, (3) Inspect the repository lock or resolution artifact to determine the exact resolved version, (4) Look up official documentation/changelog/public API reference for that exact version without relying on training-data recall, (5) Confirm every API, class, or function called exists in that exact version's documentation, (6) For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- **R-AGT-001** MUST: Inspect the repository to locate internal agent interface definitions and implement all required interface methods when introducing new agent capabilities.
- **R-AGT-002** MUST: Verify agent resolution mechanisms preserve directory-keyed scoping across both execution and revert workflows.

### Verify

```bash
# Discover the project repository test script from the package configuration and execute the complete test suite.
# Discover the repository type-checking configuration and execute compile-time interface verification.
```

**Accept when:**
- All unit and integration tests covering agent execution and reversal pass without failures.
- Static type analysis confirms complete compliance with the agent interface contract without type errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests bypassing the agent interface contract or introducing direct concrete dependencies will be blocked. Code reviews require refactoring concrete agent coupling into interface-compliant implementations before merge.
</enforcement>