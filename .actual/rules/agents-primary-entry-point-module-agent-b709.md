# AbstractAgent Core Module Adoption for Agent Implementations: Primary Entry Point Module Agent Exports

These rules are ALWAYS ACTIVE for all specialized agent implementations and module architecture integrations across the codebase.

### Rules

- **R-AGT-001** MUST: The primary entry point module for agent exports MUST re-export the unified agent contract interface and concrete agent implementations alongside AbstractAgent.
- **R-AGT-002** MUST: When introducing a new agent variant, inherit from AbstractAgent and implement all abstract lifecycle and execution methods defined by the core agent contract.
- **R-AGT-003** MUST: Export newly implemented concrete agents through the primary module aggregation file alongside existing agent definitions.

### Verify

```bash
# Discover and execute the repository's test runner to validate agent implementations
# Discover and execute the project's type-checking and linter suites
```

**Accept when:**
- All concrete agent modules inherit from AbstractAgent and pass type verification without contract mismatches.
- All agent lifecycle and execution test suites pass without regressions across all agent variants.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>