# Standardization on IAgent Interface Abstraction for Core Subsystems and Capability Providers: Concrete Agent Instances Passed Core Subsystems

These rules are ALWAYS ACTIVE for all core orchestration subsystems, revert engines, selection modules, and protocol capability providers that interact with agent instances.

### Rules

- **R-AGT-001** MUST: All concrete agent instances passed into core subsystems MUST satisfy the complete contract declared by the IAgent interface.

### Verify

```bash
# Discover the repository type check script from the configuration manifest and execute it to verify contract compliance.
# Execute the repository linting and import rule verification script to ensure no forbidden concrete agent dependencies exist in core modules.
# Run the repository automated test suite to validate agent contract execution across all dependent subsystems.
```

**Accept when:**
- Static analysis verifies that all agent consumers import and depend exclusively on the IAgent contract abstraction without referencing concrete agent classes.
- Type checking and test execution scripts defined in the repository pass with zero errors across all core and protocol capability modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent consumers must depend exclusively on the IAgent contract abstraction.
</enforcement>