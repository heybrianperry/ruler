# Adoption of Core IAgent Interface for Agent Management: Extend Iagent Interface Specialized Agent Types

These rules are ALWAYS ACTIVE for modules within src/core and src/mcp that deal with agent definition, selection, configuration, or capabilities.

### Rules

- **R-IAgent-001** MAY: Extend the IAgent interface for specialized agent types, adhering to the established contract.

### Verify

```bash
# Discover and run the project's static analysis tools.
# Discover and run the project's unit tests for agent-related modules.
# Discover and run the project's integration tests involving agent interactions.
```

**Accept when:**
- Static analysis reports no violations of IAgent interface usage.
- All agent-related unit and integration tests pass.
- New agent implementations correctly adhere to the IAgent contract.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>