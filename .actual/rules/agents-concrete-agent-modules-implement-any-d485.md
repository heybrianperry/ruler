# AbstractAgent Core Module Adoption for Agent Implementations: Concrete Agent Modules Implement Any Abstract

These rules are ALWAYS ACTIVE for all specialized agent implementations and module architecture inheritance hierarchies across the codebase.

### Rules

- **R-AGT-001** MUST: Concrete agent modules MUST implement any abstract lifecycle and execution contracts declared by AbstractAgent rather than establishing divergent execution protocols.

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