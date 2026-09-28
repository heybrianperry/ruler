# Standardization on IAgent Interface Abstraction for Core Subsystems and Capability Providers: Core Orchestration Subsystems Protocol Capability Modules

These rules are ALWAYS ACTIVE for all core orchestration subsystems, revert engines, selection modules, and protocol capability providers that interact with agent instances.

### Rules

- **R-IAG-001** MUST: Core orchestration subsystems and protocol capability modules MUST interact with agent entities exclusively through the IAgent contract abstraction.
- **R-IAG-002** MUST (Exception EXC-20-001): Factory instantiation modules specifically designated for constructing concrete agent instances are exempt from this rule.
- **R-IAG-003** MUST: Implementations interacting with agent instances must depend strictly on the properties and lifecycle methods declared on the IAgent interface contract.
- **R-IAG-004** MUST: Agent instantiation and lifecycle management must be decoupled from consumption sites through dedicated factory or selection mechanisms.

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
Claude Code MUST NOT skip or defer verification. Automated continuous integration checks, peer code reviews, and automated verification scripts enforce this rule. Pull requests introducing direct dependencies from core subsystems to concrete agent implementations are blocked.
</enforcement>