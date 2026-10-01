# Adoption of IAgent Interface Module for Agent Implementations: Before Writing Modifying Code That Integrates

These rules are ALWAYS ACTIVE for agent integration modules, agent utility routines, and agent orchestration dispatchers within the agent subsystem, and any new coding assistant or automation agent integrations being introduced to the codebase.

### Rules

- **R-AGENT-001** MUST: Before writing or modifying code that integrates versioned dependencies, the consumer MUST inspect the repository dependency manifest and authoritative lock artifact to resolve the exact installed dependency versions.
- **R-AGENT-002** MUST: All newly introduced agent providers must provide concrete implementations satisfying the IAgent contract before exposing them in the agent index export.

### Verify

```bash
# Discover and execute the repository typecheck or build script from the project manifest
# Discover and execute the repository test runner script for the agent test suite
```

**Accept when:**
- All agent integration classes strictly implement the IAgent contract with zero type checking or compilation errors.
- Agent orchestration workflows invoke diverse agent implementations solely through the IAgent contract.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration build checks verify interface conformance and static typing, and code reviews reject direct consumer imports of concrete agent implementation classes in orchestration modules.
</enforcement>