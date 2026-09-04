# Agent Base Class Architecture with AgentsMdAgent: Agent Implementations Use Centralized Filesystemutils Module

These rules are ALWAYS ACTIVE for all agent implementations in the codebase that handle configuration file access operations.

### Rules

- **R-AGENT-001** MUST: Agent implementations MUST use the centralized FileSystemUtils module for configuration file access operations rather than directly importing Node.js filesystem modules.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting rules that verify agent implementations extend the required base class
find . -name ".eslintrc*" -o -name "eslint.config.*" | head -1

# Locate the project's test suite and run tests that validate agent interface compliance and base class behavior
find . -name "*.test.ts" -o -name "*.spec.ts" | grep -i agent | head -5

# Search the agents directory for files matching the agent naming pattern and verify each imports the base class or interface
find ./agents -type f -name "*Agent.ts" -o -name "*agent.ts" 2>/dev/null | while read file; do
  echo "Checking $file for base class imports..."
  grep -E "(extends AgentsMdAgent|implements IAgent|from.*FileSystemUtils)" "$file" || echo "WARNING: $file may not use required base class or utilities"
done

# Verify no direct filesystem module imports in agent implementations
find ./agents -type f \( -name "*Agent.ts" -o -name "*agent.ts" \) 2>/dev/null | while read file; do
  if grep -q "require.*['\"]fs['\"]\|import.*from.*['\"]fs['\"]" "$file"; then
    echo "VIOLATION: $file directly imports fs module"
  fi
done
```

**Accept when:**
- All agent implementation files in the agents directory import and extend either AgentsMdAgent or implement IAgent interface
- Static analysis confirms no direct filesystem module imports in agent implementations where FileSystemUtils should be used
- All agent implementations pass interface compliance tests demonstrating they implement required base class methods
- FileSystemUtils is imported by all agent implementations that perform configuration file access operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent implementations MUST be checked for compliance with R-AGENT-001 before code is committed.
</enforcement>