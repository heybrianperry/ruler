# AgentsMdAgent Internal Module Adoption for Agent Adapters: Agent Adapters Route Common Filesystem Inspection

These rules are ALWAYS ACTIVE for all current and future agent adapter implementations in the agent integration subsystem and modules providing interfaces or base classes for AI agent interaction and markdown configuration.

### Rules

- **R-AGNT-001** SHOULD: Agent adapters SHOULD route common filesystem inspection and modification routines through shared core filesystem utilities rather than invoking low-level runtime modules directly when supported by the base module.
- **R-AGNT-002** MANDATORY: When creating a new agent adapter, inherit from the internal AgentsMdAgent class and implement all abstract lifecycle and configuration methods mandated by the IAgent contract.
- **R-AGNT-003** MANDATORY: Delegate generic configuration file parsing and markdown synchronization to base class methods, restricting adapter-specific code to tool-specific options and execution semantics.
- **R-AGNT-004** MANDATORY: Before writing code that uses a versioned library, execute in order: find dependency manifest, identify build tool, inspect repository lock or resolution artifact for exact version, look up official documentation for that exact version, confirm every API/class/function exists in that version, and re-run per dependency at point of use.

### Verify

```bash
# Discover the repository script configuration to identify and execute the static analysis and type verification suite across all agent modules.
# Discover and run the architectural linting and module boundary test suites to confirm that all agent implementations conform to the internal base module and interface contract.
```

**Accept when:**
- All agent adapter modules cleanly resolve dependencies and pass type checking while extending the internal AgentsMdAgent module.
- Module boundary checks confirm that no agent adapter module bypasses the IAgent interface or provides duplicate core agent lifecycle implementations.

<enforcement>
Verification is mandatory. Claude Code MUST NOT skip or defer verification. Pull requests containing agent adapters that do not inherit from AgentsMdAgent or implement IAgent are blocked from merging.
</enforcement>