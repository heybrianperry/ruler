# Adopt IAgent Interface Module as Centralized Agent Contract: Infrastructure Subsystems Including Protocol Handlers State

These rules are ALWAYS ACTIVE for all infrastructure code that interacts with agent abstractions, including protocol handlers, state management, configuration utilities, and agent selection logic.

### Rules

- **R-IAGENT-001** MUST: Infrastructure subsystems including protocol handlers, state management, configuration utilities, and agent selection logic MUST reference IAgent when declaring agent-typed parameters, return values, or data structures.

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
Claude Code MUST NOT skip or defer verification. Violations that introduce circular dependencies or break module boundaries are treated as high-priority defects requiring immediate remediation. Pull requests introducing direct imports of concrete agent classes in infrastructure code must be blocked until refactored to use the interface.
</enforcement>