# IAgent Core Agent Interface Module Adoption: Revert Rollback Orchestration Engines Consume Identical

These rules are ALWAYS ACTIVE for orchestration modules executing agent tasks, managing agent lifecycles, and coordinating rollback/revert routines across directory configurations.

### Rules

- **R-AGENT-001** MUST: Revert and rollback orchestration engines MUST consume identical IAgent contract abstractions and resolution mappings as primary execution engines.
- **R-AGENT-002** MANDATORY: Consumers MUST derive tool names, file names, commands, package managers, and version numbers from the project repository.
- **R-AGENT-003** MANDATORY: Consumers MUST follow the Lock-Version Grounding sequence before writing code that uses a versioned library (manifest -> build tool -> lock artifact -> official documentation -> API existence confirmation).
- **R-AGENT-004** MANDATORY: Consumers MUST inspect the repository to locate internal agent interface definitions and implement all required interface methods when introducing new agent capabilities.
- **R-AGENT-005** MANDATORY: Consumers MUST verify agent resolution mechanisms preserve directory-keyed scoping across both execution and revert workflows.

### Verify

```bash
# Discover and execute the project repository test suite using the package configuration
# Discover and execute the repository type-checking configuration for compile-time interface verification
```

**Accept when:**
- All unit and integration tests covering agent execution and reversal pass without failures.
- Static type analysis confirms complete compliance with the agent interface contract without type errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. Violation handling: Pull requests bypassing the agent interface contract or introducing direct concrete dependencies will be blocked, and code reviews require refactoring concrete agent coupling into interface-compliant implementations before merge.
</enforcement>