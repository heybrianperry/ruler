# Directory-Scoped Agent Resolution Caching with ConfigLoader and IAgent: Components Querying Agent Instances Across Directory

These rules are ALWAYS ACTIVE for operations that resolve or retrieve IAgent instances across directory-based configuration entries and service boundaries coordinating configuration loader results with agent execution contexts.

### Rules

- **R-DIR-001** MUST: Components querying agent instances across directory configurations MUST retrieve previously resolved collections from the directory-keyed map via get operations before attempting new resolutions.

### Verify

```bash
# Discover and execute the test suite declared in the project repository manifest to validate agent resolution caching.
# Run the repository static analysis and type verification scripts to ensure compliance with the IAgent interface contract.
```

**Accept when:**
- All tests verifying directory-keyed agent caching and retrieval pass successfully.
- Static type checking confirms all cached collections strictly satisfy the IAgent contract.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated test suites, peer code review, and static analysis.
</enforcement>