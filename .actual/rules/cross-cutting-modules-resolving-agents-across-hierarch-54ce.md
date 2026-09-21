# IAgent Core Agent Interface Module Adoption: Modules Resolving Agents Across Hierarchical Project

These rules are ALWAYS ACTIVE for orchestration modules executing agent tasks, managing agent lifecycles, and handling rollback/revert routines across hierarchical project structures.

### Rules

- **R-AG-001** MUST: Modules resolving agents across hierarchical project structures MUST key and retrieve resolved IAgent instances by their respective configuration root directories.

### Verify

```bash
# Discover and execute test suite and type-checking via repository configuration
# (e.g., npm test / pytest, and tsc / mypy based on repository manifest)
```

**Accept when:**
- All unit and integration tests covering agent execution and reversal pass without failures.
- Static type analysis confirms complete compliance with the agent interface contract without type errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated continuous integration checks executing repository test and type-check scripts, and peer code review.
</enforcement>