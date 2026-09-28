# Standardization on IAgent Interface Abstraction for Core Subsystems and Capability Providers: When Referencing Versioned Dependencies Associated Module

These rules are ALWAYS ACTIVE for all core orchestration subsystems, revert engines, selection modules, and protocol capability providers that interact with agent instances.

### Rules

- **R-AGENT-001** MUST: When referencing versioned dependencies associated with module interfaces, consumers MUST discover the project dependency manifest and lock artifact to resolve the authoritative version prior to implementation.
- **R-AGENT-002** MUST: Implementations interacting with agent instances must depend strictly on the properties and lifecycle methods declared on the IAgent interface contract.
- **R-AGENT-003** MUST: Agent instantiation and lifecycle management must be decoupled from consumption sites through dedicated factory or selection mechanisms.

### Verify

```bash
# Discover and execute the repository type check script from the configuration manifest
# Execute the repository linting and import rule verification script
# Run the repository automated test suite
```

**Accept when:**
- Static analysis verifies that all agent consumers import and depend exclusively on the IAgent contract abstraction without referencing concrete agent classes.
- Type checking and test execution scripts defined in the repository pass with zero errors across all core and protocol capability modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent consumers must import and depend exclusively on the IAgent contract abstraction.
</enforcement>