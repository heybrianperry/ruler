# Core Infrastructure Depends on Shared Agent Abstractions and Configuration Loader: Core Infrastructure Components Import Agent Related

These rules are ALWAYS ACTIVE for all modules within the core infrastructure package that coordinate agent behavior, require configuration access, or implement revert operations, agent selection, or other core orchestration logic.

### Rules

- **R-CORE-001** SHOULD: Core infrastructure components SHOULD import agent-related utilities from the shared agent-utils module rather than implementing utility functions locally.
- **R-CORE-002** MUST: Core infrastructure components MUST use the IAgent interface abstraction rather than importing concrete agent implementations.
- **R-CORE-003** MUST: Core infrastructure components MUST access configuration through the ConfigLoader utility rather than implementing direct configuration loading logic.
- **R-CORE-004** MUST: Dependency flow MUST remain unidirectional: core depends on agent abstractions; agents implement abstractions but do NOT depend on core.
- **R-CORE-005** SHOULD: New core infrastructure components SHOULD examine existing core modules to identify and adopt established import patterns for IAgent, ConfigLoader, and agent-utils.
- **R-CORE-006** SHOULD: Core components requiring agent coordination capabilities not provided by IAgent interface SHOULD extend the interface rather than bypassing it.

### Verify

```bash
# Discover and execute the project's static analysis tooling to verify that core package modules import IAgent interface and ConfigLoader utility
# (Tool discovery required per project configuration)

# Discover and execute the project's dependency analysis tooling to confirm unidirectional dependency flow from core to agents package
# (Tool discovery required per project configuration)

# Discover and execute the project's test suite to verify that core infrastructure components correctly use IAgent abstraction and ConfigLoader utility
# (Test runner discovery required per project configuration)
```

**Accept when:**
- Static analysis confirms all core infrastructure modules import IAgent and ConfigLoader from their respective shared module locations
- Dependency analysis confirms no circular dependencies between core and agents packages
- Dependency analysis confirms unidirectional dependency flow: core → agents (agents do not import core)
- Test suite passes with core infrastructure components successfully coordinating agents through IAgent interface and accessing configuration through ConfigLoader
- Code review verification confirms new core infrastructure components use shared abstractions

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code within scope. Violations detected by static analysis or dependency tooling MUST be resolved before merge. Exception requests require documentation and approval from architecture review team or tech lead.
</enforcement>