# Core Infrastructure Depends on Shared Agent Abstractions and Configuration Loader: Core Infrastructure Components Package Depend Iagent

These rules are ALWAYS ACTIVE for all modules within the core infrastructure package that coordinate agent behavior, require configuration access, or implement revert operations, agent selection, or other core orchestration logic.

### Rules

- **R-CORE-001** MUST: Core infrastructure components in the core package MUST depend on the IAgent interface for agent abstraction rather than concrete agent implementations.

### Verify

```bash
# Discover and execute the project's static analysis tooling to verify that core package modules import IAgent interface and ConfigLoader utility
# Discover and execute the project's dependency analysis tooling to confirm unidirectional dependency flow from core to agents package
# Discover and execute the project's test suite to verify that core infrastructure components correctly use IAgent abstraction and ConfigLoader utility
```

**Accept when:**
- Static analysis confirms all core infrastructure modules import IAgent and ConfigLoader from their respective shared module locations
- Dependency analysis confirms no circular dependencies between core and agents packages
- Test suite passes with core infrastructure components successfully coordinating agents through IAgent interface and accessing configuration through ConfigLoader

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests introducing direct agent implementation coupling in core components are rejected in code review. Continuous integration fails if static analysis detects prohibited import patterns. Violations discovered post-merge are tracked as technical debt items and prioritized for refactoring.
</enforcement>