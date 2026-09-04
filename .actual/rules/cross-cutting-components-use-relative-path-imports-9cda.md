# Adopt IAgent Interface as Canonical Type Contract for Agent Coordination: Components Use Relative Path Imports Reference

These rules are ALWAYS ACTIVE for all infrastructure components that coordinate, select, configure, or manage agent lifecycle, including MCP capability implementations, agent selection and routing logic, configuration utilities, lifecycle management components, and any infrastructure component that coordinates multiple agent types polymorphically.

### Rules

- **R-IAGENT-001** MUST: Components MUST use relative path imports to reference the IAgent interface, maintaining the established module resolution convention.

### Verify

```bash
# Discover the project's static analysis configuration and execute the type checking tool
# to verify that all infrastructure components importing IAgent have valid type references.
echo "Running type checking on infrastructure components..."

# Locate the project's module dependency analysis tooling and verify that infrastructure
# components do not import concrete agent implementation classes for type declarations.
echo "Verifying dependency graph excludes concrete agent classes in infrastructure..."

# Identify the project's linting configuration and run the linter to verify that
# relative import paths to IAgent follow the established convention.
echo "Linting relative import paths to IAgent interface..."
```

**Accept when:**
- Type checking passes with no errors related to IAgent interface usage in infrastructure components.
- Dependency analysis confirms that infrastructure components depend only on the IAgent interface, not on concrete agent implementation classes.
- All relative imports to IAgent follow the established path convention and resolve correctly.

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures block merge until infrastructure components correctly type against IAgent. Code review feedback requests refactoring of infrastructure components that import concrete agent classes. Static analysis warnings are escalated to errors for infrastructure components that violate the interface dependency rule.
</enforcement>