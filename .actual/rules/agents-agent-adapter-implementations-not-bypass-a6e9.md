# AgentsMdAgent Internal Module Adoption for Agent Adapters: Agent Adapter Implementations Not Bypass Agentsmdagent

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-ADAPTER-001** MUST_NOT: Agent adapter implementations MUST_NOT bypass the AgentsMdAgent base module by implementing independent, uncoordinated configuration formats or alternative lifecycle contracts directly.

### Verify

```bash
# Discover and run the repository script configuration for static analysis, type verification, and architectural linting across all agent modules.
# (Exact discovery command derived from repository workspace configuration)
```

**Accept when:**
- All agent adapter modules cleanly resolve dependencies and pass type checking while extending the internal AgentsMdAgent module.
- Module boundary checks confirm that no agent adapter module bypasses the IAgent interface or provides duplicate core agent lifecycle implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent adapters must derive from the AgentsMdAgent base class and implement the IAgent contract.
</enforcement>