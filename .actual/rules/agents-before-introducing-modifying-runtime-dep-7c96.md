# AbstractAgent Core Module Adoption for Agent Implementations: Before Introducing Modifying Runtime Dependencies Utilized

These rules are ALWAYS ACTIVE for all files matching the implementation and extension of specialized agent modules across the codebase.

### Rules

- **R-AGT-001** MUST: Before introducing or modifying runtime dependencies utilized by agent implementations, the consumer MUST inspect the repository lock or resolution artifact to determine and verify the exact resolved version against dependency manifests.

### Verify

```bash
# Discover and execute the repository's test runner to validate that all agent implementations successfully pass lifecycle and contract tests.
# Discover and execute the project's type-checking and linter suites to ensure all agent implementations strictly conform to AbstractAgent contracts.
```

**Accept when:**
- All concrete agent modules inherit from AbstractAgent and pass type verification without contract mismatches.
- All agent lifecycle and execution test suites pass without regressions across all agent variants.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>