# Adopt IAgent Interface as Canonical Type Contract for Agent Coordination: Components Extend Compose Iagent Interface Specialized

These rules are ALWAYS ACTIVE for all infrastructure components that coordinate, select, configure, or manage agent lifecycle, including MCP capability implementations, agent selection and routing logic, configuration utilities, lifecycle management components, and any infrastructure component that coordinates multiple agent types polymorphically.

### Rules

- **R-IAGENT-001** MAY: Components MAY extend or compose the IAgent interface for specialized coordination scenarios, provided the extension maintains compatibility with the base contract.
- **R-IAGENT-002** MUST: Infrastructure components that coordinate agents SHALL import the IAgent interface using the relative path convention observed in existing components.
- **R-IAGENT-003** MUST: Infrastructure components SHALL NOT import concrete agent implementation classes for type declarations; use the IAgent interface contract instead.
- **R-IAGENT-004** SHOULD: When an infrastructure component needs agent-specific capabilities beyond the IAgent contract, prefer type guards or capability detection patterns over importing concrete agent classes.
- **R-IAGENT-005** MUST: When modifying the IAgent interface, audit all infrastructure consumers to ensure compatibility using the project's type checking tooling.

### Verify

```bash
# Discover and run the project's static analysis configuration to verify type checking
# Verify that all infrastructure components importing IAgent have valid type references
type_checker=$(grep -r "typecheck\|type-check\|tsc" package.json tsconfig.json 2>/dev/null | head -1)
if [ -n "$type_checker" ]; then
  echo "Running type checker..."
  # Execute the discovered type checking tool
fi

# Locate and run module dependency analysis tooling
# Verify that infrastructure components do not import concrete agent implementation classes
echo "Checking for concrete agent class imports in infrastructure components..."
grep -r "from.*agents/.*Agent" --include="*.ts" --include="*.js" | grep -v "IAgent" | grep -v "test" || echo "No concrete agent imports found"

# Identify and run linting configuration
# Verify that relative import paths to IAgent follow the established convention
echo "Verifying IAgent import paths follow convention..."
grep -r "from.*IAgent" --include="*.ts" --include="*.js" | grep -v "test"
```

**Accept when:**
- Type checking passes with no errors related to IAgent interface usage in infrastructure components
- Dependency analysis confirms that infrastructure components depend only on the IAgent interface, not on concrete agent implementation classes
- All relative imports to IAgent follow the established path convention and resolve correctly
- No concrete agent implementation classes are imported in infrastructure components for type declarations

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures block merge until infrastructure components correctly type against IAgent. Code review feedback requests refactoring of infrastructure components that import concrete agent classes. Static analysis warnings are escalated to errors for infrastructure components that violate the interface dependency rule.
</enforcement>