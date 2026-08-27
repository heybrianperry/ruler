# Adopt IAgent Interface Module as Centralized Agent Contract: Infrastructure Code Import Additional Agent Related

These rules are ALWAYS ACTIVE for all infrastructure code that interacts with agent abstractions, including core infrastructure subsystems that handle agent lifecycle, coordination, or interaction; protocol layer implementations; state management systems; and configuration and selection logic that operates on agent abstractions.

### Rules

- **R-IAGENT-001** MAY: Infrastructure code MAY import additional agent-related utilities or configuration modules alongside IAgent when those modules provide complementary functionality without violating the interface abstraction boundary.

- **R-IAGENT-002** MUST: All core infrastructure modules (protocol handlers, state management, configuration, selection logic) that reference agent contracts MUST import the centralized IAgent interface module rather than concrete agent implementation classes.

- **R-IAGENT-003** MUST NOT: Infrastructure code MUST NOT import concrete agent implementation classes where only the interface contract is required; direct imports of concrete agent classes break the abstraction boundary and introduce coupling.

- **R-IAGENT-004** SHOULD: When infrastructure code needs agent-specific behavior beyond the interface contract, developers SHOULD extend the interface with capability queries rather than importing concrete classes, preserving the abstraction boundary.

### Verify

```bash
# Discover the project's module analysis or import checking tooling and execute it
# to verify that infrastructure modules import the interface module rather than concrete agent classes
find . -name "*.md" -o -name "*.json" | xargs grep -l "import.*analysis\|linting" | head -1

# Locate the project's static analysis or linting configuration and run configured checks
# to detect violations of the interface import pattern in core infrastructure code
find . -name ".eslintrc*" -o -name "pylint*" -o -name "ruff.toml" -o -name "pyproject.toml" | head -1

# Identify the project's test suite and execute tests covering the 4 observed infrastructure subsystems
# (MCP capabilities, revert engine, config utilities, agent selection) to confirm they interact with agents through the interface contract
find . -type f \( -name "*test*" -o -name "*spec*" \) | grep -E "(mcp|revert|config|select)" | head -5

# Search for direct imports of concrete agent classes in infrastructure code
grep -r "from.*agents.*import" --include="*.ts" --include="*.js" --include="*.py" | grep -v "IAgent" | grep -v "test" | grep -v "spec"
```

**Accept when:**
- All core infrastructure modules (protocol handlers, state management, configuration, selection logic) import the interface module when referencing agent contracts
- No direct imports of concrete agent implementation classes exist in infrastructure code where only the interface contract is required
- Static analysis or linting checks pass, confirming adherence to the interface import pattern across all infrastructure subsystems
- Tests covering the 4 observed infrastructure subsystems (MCP capabilities, revert engine, config utilities, agent selection) pass and confirm agents are accessed through the interface contract

<enforcement>
Claude Code MUST NOT skip or defer verification. All infrastructure code modifications MUST be checked against R-IAGENT-002 and R-IAGENT-003 before acceptance. Pull requests introducing direct imports of concrete agent classes in infrastructure code MUST be blocked until refactored to use the interface.
</enforcement>