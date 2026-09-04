# Agent Base Class Architecture with AgentsMdAgent: Agent Implementations Placed Agents Directory Follow

These rules are ALWAYS ACTIVE for all agent implementations in the agents directory that require consistent behavior, interface contracts, and configuration file access patterns.

### Rules

- **R-AGENT-001** MUST: Agent implementations MUST be placed in the agents directory and follow the naming convention of the agent type followed by 'Agent' suffix.
- **R-AGENT-002** MUST: Agent implementations MUST extend either AgentsMdAgent or implement the IAgent interface.
- **R-AGENT-003** MUST: All configuration file operations MUST use the FileSystemUtils module for path resolution, file existence checks, and content reading.
- **R-AGENT-004** SHOULD: New agent implementations should examine the base class implementation to understand required interface methods and available utility functions before writing agent-specific logic.
- **R-AGENT-005** SHOULD: Agent implementations should use relative imports to maintain module cohesion and avoid circular dependencies.

### Verify

```bash
# Discover the project's static analysis configuration and execute linting rules
# that verify agent implementations extend the required base class
find ./agents -name '*Agent.js' -o -name '*Agent.ts' | while read file; do
  grep -E '(extends AgentsMdAgent|implements IAgent)' "$file" || echo "FAIL: $file missing base class"
done

# Verify no direct filesystem module imports in agent implementations
find ./agents -name '*Agent.js' -o -name '*Agent.ts' | while read file; do
  if grep -q "require('fs')\|from 'fs'\|require('path')\|from 'path'" "$file"; then
    if ! grep -q 'FileSystemUtils' "$file"; then
      echo "FAIL: $file uses fs/path directly instead of FileSystemUtils"
    fi
  fi
done

# Run project test suite to validate agent interface compliance
npm test -- --testPathPattern=agent
```

**Accept when:**
- All agent implementation files in the agents directory import and extend either AgentsMdAgent or implement IAgent interface
- Static analysis confirms no direct filesystem module imports in agent implementations where FileSystemUtils should be used
- All agent implementations pass interface compliance tests demonstrating they implement required base class methods
- Agent files follow the naming convention of agent type followed by 'Agent' suffix

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent implementations must satisfy these rules before code is committed.
</enforcement>