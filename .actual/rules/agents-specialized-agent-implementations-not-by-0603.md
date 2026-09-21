# AbstractAgent Core Module Adoption for Agent Implementations: Specialized Agent Implementations Not Bypass Abstractagent

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-AGT-001** MUST_NOT: Specialized agent implementations MUST NOT bypass AbstractAgent by implementing standalone agent structures disconnected from the shared base contract.

### Verify

```bash
# Discover and execute the repository's test runner to validate agent implementations
# Discover and execute the project's type-checking and linter suites
```

**Accept when:**
- All concrete agent modules inherit from AbstractAgent and pass type verification without contract mismatches.
- All agent lifecycle and execution test suites pass without regressions across all agent variants.

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent implementations must inherit from AbstractAgent and conform to core contracts.
</enforcement>