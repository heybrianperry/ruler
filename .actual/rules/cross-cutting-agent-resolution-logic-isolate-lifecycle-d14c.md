# IAgent Core Agent Interface Module Adoption: Agent Resolution Logic Isolate Lifecycle Initialization

These rules are ALWAYS ACTIVE for all orchestration modules executing agent tasks, managing agent lifecycles across directory configurations, and coordinating agent-specific reversal or rollback operations.

### Rules

- **R-AGENT-001** SHOULD: Agent resolution logic SHOULD isolate agent lifecycle initialization from downstream operation invocation.

### Verify

```bash
# Discover and execute test suite and type-checking via project repository configuration
# (Repository-specific test and type-check commands must be discovered from package configuration)
```

**Accept when:**
- All unit and integration tests covering agent execution and reversal pass without failures.
- Static type analysis confirms complete compliance with the agent interface contract without type errors.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>