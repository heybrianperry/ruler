# AbstractAgent Core Module Adoption for Agent Implementations: Agent Modules Perform Environment Filesystem Resolution

These rules are ALWAYS ACTIVE for the implementation and extension of specialized agent modules and their environment/filesystem resolution logic across the codebase.

### Rules

- **R-AGT-001** SHOULD: Agent modules SHOULD perform environment and filesystem resolution using standard path utility modules when computing localized configurations or working directories.

### Verify

```bash
# Discover and execute the repository's test runner to validate that all agent implementations successfully pass lifecycle and contract tests.
# Discover and execute the project's type-checking and linter suites to ensure all agent implementations strictly conform to AbstractAgent contracts.
```

**Accept when:**
- All concrete agent modules inherit from AbstractAgent and pass type verification without contract mismatches.
- All agent lifecycle and execution test suites pass without regressions across all agent variants.

<enforcement>
Claude Code MUST NOT skip or defer verification. All concrete agent implementations must correctly perform filesystem and environment resolution using standard path utilities and extend AbstractAgent.
</enforcement>