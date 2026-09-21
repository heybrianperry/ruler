# AgentsMdAgent Module Adoption for Agent Implementations: Agent Integration Modules Not Implement Independent

These rules are ALWAYS ACTIVE for development and maintenance of agent integration modules providing tool-specific or environment-specific agent workflows, and shared agent configuration, document generation, and interface definitions within the agent subsystem.

### Rules

- **R-AGT-001** MUST_NOT: Agent integration modules MUST NOT implement independent or custom markdown configuration parsers that duplicate functionality established within AgentsMdAgent.

### Verify

```bash
# Discover the project test runner configuration from the repository manifest and execute the test suite covering agent integration modules.
# Discover the project static analysis and linting configuration from the repository manifest and execute validation across all agent integration components.
```

**Accept when:**
- All agent integration modules successfully instantiate and inherit from AgentsMdAgent without contract violations.
- All agent test suites pass cleanly with zero linting and type-checking diagnostics reported across agent implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory.
</enforcement>