# Adopt IAgent Interface Module as Centralized Agent Contract: New Infrastructure Components That Need Interact

These rules are ALWAYS ACTIVE for all core infrastructure subsystems that interact with agent abstractions, including protocol handlers, state management systems, configuration utilities, and agent selection logic.

### Rules

- **R-IAGENT-001** SHOULD: New infrastructure components that need to interact with agents SHOULD follow the established import pattern observed in MCP capabilities, revert engine, config utilities, and agent selection modules by importing from the centralized IAgent interface module rather than concrete agent implementations.

### Verify

```bash
# Discover the project's module analysis or import checking tooling and verify infrastructure modules import the interface module
find . -type f -name '*.ts' -o -name '*.js' | xargs grep -l 'infrastructure\|protocol\|state.*management\|config\|selection' | xargs grep -E 'from.*agents' | grep -v 'IAgent'

# Locate the project's static analysis or linting configuration and run configured checks
# (Command varies by project; check package.json scripts or build configuration)

# Identify and execute the project's test suite for infrastructure subsystems
# (Command varies by project; typically npm test, yarn test, or similar)
```

**Accept when:**
- All core infrastructure modules (protocol handlers, state management, configuration, selection logic) import the interface module when referencing agent contracts
- No direct imports of concrete agent implementation classes exist in infrastructure code where only the interface contract is required
- Static analysis or linting checks pass, confirming adherence to the interface import pattern across all infrastructure subsystems
- Tests covering the 4 observed infrastructure subsystems pass, confirming they interact with agents through the interface contract

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests introducing direct imports of concrete agent classes in infrastructure code MUST be blocked until refactored to use the interface. Violations that introduce circular dependencies or break module boundaries are treated as high-priority defects requiring immediate remediation.
</enforcement>