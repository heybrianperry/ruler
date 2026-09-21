# AbstractAgent Core Module Adoption for Agent Implementations: Specialized Agent Implementations Override Hook Methods

These rules are ALWAYS ACTIVE for all specialized agent implementations and module inheritance hierarchies across the codebase.

### Rules

- **R-AGT-001** MAY: Specialized agent implementations MAY override hook methods provided by AbstractAgent to supply provider-specific communication or configuration logic.

### Verify

```bash
# Discover and execute the repository's test runner to validate lifecycle and contract tests
# Discover and execute the project's type-checking and linter suites
```

**Accept when:**
- All concrete agent modules inherit from AbstractAgent and pass type verification without contract mismatches.
- All agent lifecycle and execution test suites pass without regressions across all agent variants.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated static analysis, type checking, and peer code review enforce these rules, and pull requests failing contract conformance are blocked from merging.
</enforcement>