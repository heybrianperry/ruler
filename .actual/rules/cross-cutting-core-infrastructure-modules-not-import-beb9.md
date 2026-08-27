# Adopt IAgent Interface Module as Centralized Agent Contract: Core Infrastructure Modules Not Import Concrete

These rules are ALWAYS ACTIVE for all core infrastructure modules that interact with agent abstractions, including protocol handlers, state management systems, configuration utilities, and agent selection logic.

### Rules

- **R-IAGENT-001** MUST NOT: Core infrastructure modules MUST NOT import concrete agent implementation classes when only the interface contract is required for type checking or polymorphic handling.

### Verify

```bash
# Discover the project's module analysis or import checking tooling from the repository and execute it to verify that infrastructure modules import the interface module rather than concrete agent classes

# Locate the project's static analysis or linting configuration and run the configured checks to detect any violations of the interface import pattern in core infrastructure code

# Identify the project's test suite and execute tests covering the 4 observed infrastructure subsystems to confirm they interact with agents through the interface contract
```

**Accept when:**
- All core infrastructure modules (protocol handlers, state management, configuration, selection logic) import the interface module when referencing agent contracts
- No direct imports of concrete agent implementation classes exist in infrastructure code where only the interface contract is required
- Static analysis or linting checks pass, confirming adherence to the interface import pattern across all infrastructure subsystems

<enforcement>
Verification by static analysis or linting rules that detect imports of concrete agent classes in infrastructure modules is mandatory. Code review checklist verification that new infrastructure code imports the interface module rather than concrete implementations is mandatory. Automated import analysis in continuous integration that flags violations of the interface abstraction boundary is mandatory. Claude Code MUST NOT skip or defer verification.
</enforcement>