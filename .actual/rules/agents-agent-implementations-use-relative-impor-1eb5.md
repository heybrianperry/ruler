# Agent Base Class Architecture with AgentsMdAgent: Agent Implementations Use Relative Imports Reference

These rules are ALWAYS ACTIVE for all agent implementation files in the agents directory that require consistent behavior, interface contracts, and configuration file access patterns.

### Rules

- **R-AGENT-001** SHOULD: Agent implementations SHOULD use relative imports to reference the base classes and utility modules to maintain module cohesion.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting rules that verify agent implementations extend the required base class
find . -name ".eslintrc*" -o -name "tsconfig.json" -o -name "package.json" | head -1

# Locate the project's test suite and run tests that validate agent interface compliance and base class behavior
find . -path "*/test*" -name "*.test.*" -o -name "*.spec.*" | head -5

# Search the agents directory for files matching the agent naming pattern and verify each imports the base class or interface
find ./agents -type f -name "*Agent.ts" -o -name "*Agent.js" | while read file; do grep -l "from.*AgentsMdAgent\|from.*IAgent\|import.*AgentsMdAgent\|import.*IAgent" "$file" || echo "Missing base class import: $file"; done

# Verify relative imports are used in agent implementations
find ./agents -type f \( -name "*Agent.ts" -o -name "*Agent.js" \) -exec grep -l "from \"\./\|from '\./" {} \;
```

**Accept when:**
- All agent implementation files in the agents directory import and extend either AgentsMdAgent or implement IAgent interface
- Static analysis confirms no direct filesystem module imports in agent implementations where FileSystemUtils should be used
- All agent implementations pass interface compliance tests demonstrating they implement required base class methods
- Agent implementations use relative imports (e.g., `from './base'` or `from '../utils'`) to reference base classes and utility modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent implementations MUST be checked for base class imports and relative import usage before code is committed.
</enforcement>