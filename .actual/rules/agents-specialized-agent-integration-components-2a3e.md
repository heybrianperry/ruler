# AgentsMdAgent Module Adoption for Agent Implementations: Specialized Agent Integration Components Adopt Internal

These rules are ALWAYS ACTIVE for all specialized agent integration components and agent subsystem modules.

### Rules

- **R-AGT-001** MUST: All specialized agent integration components MUST adopt the internal AgentsMdAgent module as their core base implementation for managing agent configuration documents and operational workflows.

### Verify

```bash
# Discover the project test runner configuration from the repository manifest and execute the test suite covering agent integration modules.
# Discover the project static analysis and linting configuration from the repository manifest and execute validation across all agent integration components.
```

**Accept when:**
- All agent integration modules successfully instantiate and inherit from AgentsMdAgent without contract violations.
- All agent test suites pass cleanly with zero linting and type-checking diagnostics reported across agent implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration pipelines and peer code review validate that any new or modified agent integration extends AgentsMdAgent and adheres to the IAgent interface.
</enforcement>