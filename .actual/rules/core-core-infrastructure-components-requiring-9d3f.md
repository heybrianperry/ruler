# Core Infrastructure Depends on Shared Agent Abstractions and Configuration Loader: Core Infrastructure Components Requiring Configuration Access

These rules are ALWAYS ACTIVE for all core infrastructure components that coordinate agent behavior or require configuration access.

### Rules

- **R-CORE-001** MUST: Core infrastructure components requiring configuration access MUST use the ConfigLoader utility module.
- **R-CORE-002** MUST: Core infrastructure components MUST depend on the IAgent interface abstraction rather than concrete agent implementations.
- **R-CORE-003** MUST: Core infrastructure components MUST maintain unidirectional dependency flow: core depends on agent abstractions, agents implement abstractions but do not depend on core.
- **R-CORE-004** SHOULD: New core infrastructure components SHOULD examine existing core modules to identify and adopt established import patterns for IAgent, ConfigLoader, and agent-utils.
- **R-CORE-005** SHOULD: Core components requiring agent coordination capabilities not provided by IAgent interface SHOULD extend the interface rather than bypass it.

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
- Dependency analysis confirms unidirectional dependency flow from core to agents package
- Test suite passes with core infrastructure components successfully coordinating agents through IAgent interface and accessing configuration through ConfigLoader
- Code review verifies new core infrastructure components use shared abstractions

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis in continuous integration MUST detect direct imports of concrete agent implementations in core package. Dependency graph analysis MUST detect circular dependencies or violations of unidirectional dependency flow. Pull requests introducing direct agent implementation coupling in core components MUST be rejected in code review.
</enforcement>