# Standardization on IAgent Interface Abstraction for Core Subsystems and Capability Providers: New Agent Capabilities Operational Behaviors Defined

These rules are ALWAYS ACTIVE for all core orchestration subsystems, revert engines, selection modules, and protocol capability providers that interact with agent instances.

### Rules

- **R-AGENT-001** SHOULD: New agent capabilities and operational behaviors SHOULD be defined as additions or extensions to the IAgent interface rather than proprietary subsystem interfaces.

### Verify

```bash
# Discover and run the repository type check script from the configuration manifest
# Execute repository linting and import rule verification scripts
# Run the repository automated test suite
```

**Accept when:**
- Static analysis verifies that all agent consumers import and depend exclusively on the IAgent contract abstraction without referencing concrete agent classes.
- Type checking and test execution scripts defined in the repository pass with zero errors across all core and protocol capability modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration checks and mandatory peer reviews enforce that core subsystems consume agent abstractions exclusively through the IAgent contract.
</enforcement>