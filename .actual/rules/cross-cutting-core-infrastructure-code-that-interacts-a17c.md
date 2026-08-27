# Adopt IAgent Interface Module as Centralized Agent Contract: Core Infrastructure Code That Interacts Agent

These rules are ALWAYS ACTIVE for all core infrastructure code that interacts with agent abstractions, including protocol handlers, state management systems, configuration utilities, and agent selection logic.

### Rules

- **R-IAGENT-001** MUST: All core infrastructure code that interacts with agent abstractions MUST import the IAgent interface module as the contract definition rather than depending on concrete agent implementation classes.

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
Verification by static analysis, code review checklist, and automated import analysis in continuous integration is mandatory. Pull requests introducing direct imports of concrete agent classes in infrastructure code must be blocked until refactored to use the interface. Violations are treated as technical debt or high-priority defects depending on severity. Exceptions require documented justification and must be recorded with expiration dates.
</enforcement>