# AgentsMdAgent Internal Module Adoption for Agent Adapters: Prior Introducing Utilizing Any External Parsing

These rules are ALWAYS ACTIVE for all current and future agent adapter implementations in the agent integration subsystem, and modules providing interfaces or base classes for AI agent interaction and markdown configuration.

### Rules

- **R-AGENTS-001** MUST: Prior to introducing or utilizing any external parsing or serialization dependencies for agent configuration, the consumer MUST inspect the project lock artifact to resolve and verify the exact dependency version against documented interface contracts.
- **R-AGENTS-002** MANDATORY: Execute in order for lock-version grounding before writing code that uses a versioned library: (1) Find dependency manifest, (2) Identify build tool, (3) Inspect repository lock/resolution artifact for exact version, (4) Look up official documentation for that exact version, (5) Confirm every API/class/function exists in that version, (6) Re-run steps 3-5 per dependency at point of use.
- **R-AGENTS-003** MANDATORY: When creating a new agent adapter, inherit from the internal AgentsMdAgent class and implement all abstract lifecycle and configuration methods mandated by the IAgent contract.
- **R-AGENTS-004** MANDATORY: Delegate generic configuration file parsing and markdown synchronization to base class methods, restricting adapter-specific code to tool-specific options and execution semantics.

### Verify

```bash
# Discover and execute the static analysis and type verification suite across all agent modules
# Discover and run the architectural linting and module boundary test suites
```

**Accept when:**
- All agent adapter modules cleanly resolve dependencies and pass type checking while extending the internal AgentsMdAgent module.
- Module boundary checks confirm that no agent adapter module bypasses the IAgent interface or provides duplicate core agent lifecycle implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests containing agent adapters that do not inherit from AgentsMdAgent or implement IAgent are blocked from merging.
</enforcement>