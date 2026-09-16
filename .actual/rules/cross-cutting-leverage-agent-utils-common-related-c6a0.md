# Adoption of Core IAgent Interface for Agent Management: Leverage Agent Utils Common Related Utility

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- R-AGENTUTIL-001 SHOULD: leverage agent-utils for common agent-related utility functions.

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