# Core Infrastructure Depends on Shared Agent Abstractions and Configuration Loader: Core Infrastructure Components Not Bypass Iagent

These rules are ALWAYS ACTIVE for all modules within the core infrastructure package that coordinate agent behavior, require configuration access, or implement revert operations, agent selection, or other core orchestration logic.

### Rules

- **R-CORE-001** MUST NOT: Core infrastructure components MUST NOT bypass the IAgent interface to directly couple with concrete agent implementation classes.

### Verify

```bash
# Discover and execute the project's static analysis tooling to verify that core package modules import IAgent interface and ConfigLoader utility
# (Tool discovery required from project repository)

# Discover and execute the project's dependency analysis tooling to confirm unidirectional dependency flow from core to agents package
# (Tool discovery required from project repository)

# Discover and execute the project's test suite to verify that core infrastructure components correctly use IAgent abstraction and ConfigLoader utility
# (Test command discovery required from project repository)
```

**Accept when:**
- Static analysis confirms all core infrastructure modules import IAgent and ConfigLoader from their respective shared module locations
- Dependency analysis confirms no circular dependencies between core and agents packages
- Test suite passes with core infrastructure components successfully coordinating agents through IAgent interface and accessing configuration through ConfigLoader

<enforcement>
Claude Code MUST NOT skip or defer verification. All core infrastructure components must be validated against this rule before code is committed.
</enforcement>