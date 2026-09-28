# AgentsMdAgent Internal Module Adoption for Agent Adapters: Every Agent Adapter Module Import Extend

These rules are ALWAYS ACTIVE for all agent adapter implementations and related AI agent interaction modules in the agent integration subsystem.

### Rules

- **R-AGNT-001** MUST: Every agent adapter module MUST import and extend the internal AgentsMdAgent base module and implement the IAgent interface contract to standardize agent configuration and execution lifecycles.

### Verify

```bash
# Discover and run the repository's static analysis, type verification, and architectural lint suites
# (Commands must be discovered from the repository script configuration)
```

**Accept when:**
- All agent adapter modules cleanly resolve dependencies and pass type checking while extending the internal AgentsMdAgent module.
- Module boundary checks confirm that no agent adapter module bypasses the IAgent interface or provides duplicate core agent lifecycle implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration checks and architectural reviews strictly verify that all agent adapters derive from the AgentsMdAgent base class and implement the IAgent interface.
</enforcement>