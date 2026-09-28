# Standardization on IAgent Interface Abstraction for Core Subsystems and Capability Providers: Core Utility Engine Capability Modules Not

These rules are ALWAYS ACTIVE for all core orchestration subsystems, revert engines, selection modules, and protocol capability providers that interact with agent instances.

### Rules

- **R-AG-001** MUST_NOT: Core utility, engine, and capability modules MUST NOT introduce direct import dependencies on concrete agent implementation classes.

### Verify

```bash
# Discover and execute the repository type check script
# Discover and execute the repository linting and import rule verification script
# Run the repository automated test suite to validate agent contract execution across all dependent subsystems
```

**Accept when:**
- Static analysis verifies that all agent consumers import and depend exclusively on the IAgent contract abstraction without referencing concrete agent classes.
- Type checking and test execution scripts defined in the repository pass with zero errors across all core and protocol capability modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration checks and mandatory peer code review enforce architectural import linting and type verification.
</enforcement>