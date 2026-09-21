# IAgent Core Agent Interface Module Adoption: Core Orchestration Pipelines Interacting Agents Program

These rules are ALWAYS ACTIVE for all core orchestration pipelines interacting with agents, including forward execution modules and rollback/revert routines managing agent lifecycles across directory configurations.

### Rules

- **R-AGENT-001** MUST: All core orchestration pipelines interacting with agents MUST program against the shared IAgent interface contract rather than concrete agent implementations.

### Verify

```bash
# Discover the test script from the package configuration and execute the complete test suite
# Discover the type-checking configuration and execute compile-time interface verification
```

**Accept when:**
- All unit and integration tests covering agent execution and reversal pass without failures.
- Static type analysis confirms complete compliance with the agent interface contract without type errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration checks and peer code reviews verify adherence to the shared agent interface contract and directory-keyed scoping.
</enforcement>