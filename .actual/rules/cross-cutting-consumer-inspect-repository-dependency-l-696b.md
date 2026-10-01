# Directory-Scoped Agent Resolution Caching with ConfigLoader and IAgent: Consumer Inspect Repository Dependency Lock Artifact

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-DIR-001** MUST: The consumer MUST inspect the repository dependency lock artifact and confirm that all core runtime modules and interfaces match the recorded resolved versions before modifying service boundary contracts.

### Verify

```bash
# Discover and execute the test suite declared in the project repository manifest to validate agent resolution caching
# Run the repository static analysis and type verification scripts to ensure compliance with the IAgent interface contract
```

**Accept when:**
- All tests verifying directory-keyed agent caching and retrieval pass successfully.
- Static type checking confirms all cached collections strictly satisfy the IAgent contract.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory.
</enforcement>