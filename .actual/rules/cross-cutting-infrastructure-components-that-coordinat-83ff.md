# Adopt IAgent Interface as Canonical Type Contract for Agent Coordination: Infrastructure Components That Coordinate Select Configure

These rules are ALWAYS ACTIVE for all infrastructure components that coordinate, select, configure, or manage agent lifecycle, including MCP capability implementations, agent selection and routing logic, configuration utilities, lifecycle management components, and any infrastructure component that coordinates multiple agent types polymorphically.

### Rules

- **R-IAGENT-001** MUST: Infrastructure components that coordinate, select, configure, or manage agent lifecycle MUST import and type agent references against the IAgent interface from the agents module.

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
if command -v depcheck &> /dev/null; then
  depcheck --ignores="@types/*" .
fi

# Identify the project's linting configuration and run the linter to verify that
# relative import paths to IAgent follow the established convention.
if [ -f ".eslintrc" ] || [ -f ".eslintrc.json" ] || [ -f "eslint.config.js" ]; then
  npx eslint . --rule "no-restricted-imports: off" || yarn eslint . || pnpm eslint .
fi

# Verify no concrete agent class imports in infrastructure components
grep -r "from.*agents/.*Agent" --include="*.ts" --include="*.js" | grep -v "IAgent" && exit 1 || exit 0
```

**Accept when:**
- Type checking passes with no errors related to IAgent interface usage in infrastructure components.
- Dependency analysis confirms that infrastructure components depend only on the IAgent interface, not on concrete agent implementation classes.
- All relative imports to IAgent follow the established path convention and resolve correctly.
- No concrete agent implementation classes are imported for type declarations in infrastructure components.

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures block merge until infrastructure components correctly type against IAgent. Code review feedback requests refactoring of infrastructure components that import concrete agent classes. Static analysis warnings are escalated to errors for infrastructure components that violate the interface dependency rule.
</enforcement>