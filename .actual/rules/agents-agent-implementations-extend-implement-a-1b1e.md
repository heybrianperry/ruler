# Agent Base Class Architecture with AgentsMdAgent: Agent Implementations Extend Implement Agentsmdagent Base

These rules are ALWAYS ACTIVE for all agent implementations in the codebase that handle configuration file access, read filesystem resources, or participate in the agent architecture.

### Rules

- **R-AGENT-001** MUST: All agent implementations MUST extend or implement the AgentsMdAgent base class or IAgent interface to ensure consistent agent behavior and interface contracts.
- **R-AGENT-002** MUST: All agent implementations MUST use the FileSystemUtils module for configuration file operations including path resolution, file existence checks, and content reading.
- **R-AGENT-003** MUST: Agent implementations MUST NOT directly import Node.js filesystem modules when FileSystemUtils provides equivalent functionality.
- **R-AGENT-004** SHOULD: New agent implementations SHOULD examine the base class implementation to understand required interface methods and available utility functions before writing agent-specific logic.
- **R-AGENT-005** SHOULD: Agent implementations SHOULD use relative imports to maintain module cohesion and avoid circular dependencies.

### Verify

```bash
# Discover the project's static analysis configuration and execute linting rules
# that verify agent implementations extend the required base class
find . -name '.eslintrc*' -o -name 'tsconfig.json' -o -name '.prettierrc*' | head -1

# Search the agents directory for files matching the agent naming pattern
find ./agents -type f -name '*Agent.ts' -o -name '*Agent.js' 2>/dev/null | sort

# Verify each agent file imports the base class or interface
for file in $(find ./agents -type f \( -name '*Agent.ts' -o -name '*Agent.js' \) 2>/dev/null); do
  echo "Checking $file:"
  grep -E '(extends AgentsMdAgent|implements IAgent|from.*AgentsMdAgent|from.*IAgent)' "$file" || echo "  WARNING: No base class import found"
done

# Verify no direct filesystem imports in agent implementations
for file in $(find ./agents -type f \( -name '*Agent.ts' -o -name '*Agent.js' \) 2>/dev/null); do
  if grep -q "require('fs')\|from 'fs'\|require('path')\|from 'path'" "$file"; then
    echo "WARNING: Direct filesystem import in $file - should use FileSystemUtils"
  fi
done

# Locate and run the project's test suite for agent interface compliance
if [ -f 'package.json' ]; then
  grep -q '"test"' package.json && npm test -- --testPathPattern=agent 2>/dev/null || echo "Test command not found"
fi
```

**Accept when:**
- All agent implementation files in the agents directory import and extend either AgentsMdAgent or implement IAgent interface
- Static analysis confirms no direct filesystem module imports in agent implementations where FileSystemUtils should be used
- All agent implementations pass interface compliance tests demonstrating they implement required base class methods
- New agent implementations follow the established pattern of importing the base class using relative imports

<enforcement>
Clause MUST NOT skip or defer verification. All agent implementations MUST satisfy R-AGENT-001 and R-AGENT-002 before code review approval. Violations block continuous integration pipeline execution until base class imports and FileSystemUtils usage are corrected.
</enforcement>