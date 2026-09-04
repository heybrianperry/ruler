# Adopt IAgent Interface as Canonical Type Contract for Agent Coordination: New Infrastructure Components That Interact Agents

These rules are ALWAYS ACTIVE for all infrastructure components that coordinate, select, configure, or manage agent lifecycle, including MCP capability implementations, agent selection and routing logic, configuration utilities, lifecycle management components, and any infrastructure component that coordinates multiple agent types polymorphically.

### Rules

- **R-IAGENT-001** SHOULD: New infrastructure components that interact with agents SHOULD follow the established pattern of co-importing IAgent alongside related infrastructure modules such as configuration loaders and utility functions.
- **R-IAGENT-002** MUST: When creating new infrastructure components that coordinate agents, import the IAgent interface using the relative path convention observed in existing components and co-locate the import with other infrastructure dependencies for consistency.
- **R-IAGENT-003** SHOULD: If an infrastructure component needs agent-specific capabilities beyond the IAgent contract, consider using type guards or capability detection patterns rather than importing concrete agent classes.
- **R-IAGENT-004** MUST: When modifying the IAgent interface, audit all infrastructure consumers to ensure compatibility and use the project's type checking tooling to identify breaking changes before committing.

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
if [ -f ".eslintrc" ] || [ -f ".eslintrc.json" ] || [ -f ".eslintrc.js" ]; then
  npx eslint . --rule "no-restricted-imports: error" || yarn eslint . || pnpm eslint .
fi
```

**Accept when:**
- Type checking passes with no errors related to IAgent interface usage in infrastructure components.
- Dependency analysis confirms that infrastructure components depend only on the IAgent interface, not on concrete agent implementation classes.
- All relative imports to IAgent follow the established path convention and resolve correctly.
- Code review verification confirms new infrastructure components follow the IAgent import pattern.
- Static analysis tooling detects no imports of concrete agent classes in infrastructure components.

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures block merge until infrastructure components correctly type against IAgent. Code review feedback requests refactoring of infrastructure components that import concrete agent classes. Static analysis warnings are escalated to errors for infrastructure components that violate the interface dependency rule. Exceptions require architectural review and documentation of why the IAgent contract is insufficient for the use case.
</enforcement>