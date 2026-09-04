# Adopt IAgent Interface as Canonical Type Contract for Agent Coordination: Infrastructure Components Not Import Concrete Agent

These rules are ALWAYS ACTIVE for all infrastructure components that coordinate, select, configure, or manage agent lifecycle, including MCP capability implementations, agent selection and routing logic, configuration utilities, lifecycle management components, and any infrastructure component that coordinates multiple agent types polymorphically.

### Rules

- **R-IAGENT-001** MUST NOT: Infrastructure components MUST NOT import concrete agent implementation classes for type declarations; they MUST depend only on the IAgent interface contract.

### Verify

```bash
# Discover the project's static analysis configuration and execute the type checking tool
# to verify that all infrastructure components importing IAgent have valid type references.
type_checker=$(grep -r "typecheck\|type-check\|tsc" package.json 2>/dev/null | head -1)
if [ -n "$type_checker" ]; then
  npm run typecheck || yarn typecheck || pnpm typecheck
fi

# Locate the project's module dependency analysis tooling and verify that infrastructure
# components do not import concrete agent implementation classes for type declarations.
grep -r "import.*from.*agent/" .actual/infrastructure/ 2>/dev/null | grep -v "IAgent" && echo "FAIL: Found concrete agent imports" || echo "PASS: No concrete agent imports detected"

# Identify the project's linting configuration and run the linter to verify that
# relative import paths to IAgent follow the established convention.
lint_cmd=$(grep -r "lint" package.json 2>/dev/null | head -1)
if [ -n "$lint_cmd" ]; then
  npm run lint || yarn lint || pnpm lint
fi
```

**Accept when:**
- Type checking passes with no errors related to IAgent interface usage in infrastructure components.
- Dependency analysis confirms that infrastructure components depend only on the IAgent interface, not on concrete agent implementation classes.
- All relative imports to IAgent follow the established path convention and resolve correctly.

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures block merge until infrastructure components correctly type against IAgent. Code review feedback requests refactoring of infrastructure components that import concrete agent classes. Static analysis warnings are escalated to errors for infrastructure components that violate the interface dependency rule.
</enforcement>