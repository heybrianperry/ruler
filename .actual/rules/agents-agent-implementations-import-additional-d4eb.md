# Agent Base Class Architecture with AgentsMdAgent: Agent Implementations Import Additional Third Party

These rules are ALWAYS ACTIVE for all agent implementations in the codebase that handle configuration file access and participate in the agent architecture.

### Rules

- **R-AGENT-001** MAY: Agent implementations MAY import additional third-party parsing libraries for specific configuration formats when the base utilities do not provide the required functionality.
- **R-AGENT-002** MUST: All new agent implementations added to the agents directory MUST extend either AgentsMdAgent or implement the IAgent interface.
- **R-AGENT-003** MUST: Agent implementations MUST use the FileSystemUtils module for all configuration file operations including path resolution, file existence checks, and content reading.
- **R-AGENT-004** SHOULD: Agent implementations SHOULD follow the established pattern of importing the base class and utilities using relative imports to maintain module cohesion and avoid circular dependencies.
- **R-AGENT-005** MUST: When implementing a new agent, developers MUST examine the base class implementation to understand the required interface methods and available utility functions before writing agent-specific logic.

### Verify

```bash
# Discover the project's static analysis configuration and execute linting rules
# that verify agent implementations extend the required base class
find . -name '.eslintrc*' -o -name 'tsconfig.json' -o -name '.prettierrc*' | head -1

# Locate and run the project's test suite for agent interface compliance
find . -name 'package.json' -exec grep -l '"test"' {} \; | head -1

# Search the agents directory for files matching the agent naming pattern
find ./agents -type f -name '*Agent.ts' -o -name '*Agent.js' 2>/dev/null | sort

# Verify each agent file imports the base class or interface
grep -l 'AgentsMdAgent\|IAgent' ./agents/*Agent.* 2>/dev/null | wc -l

# Verify no direct filesystem module imports in agent implementations
grep -n "require('fs')\|from 'fs'\|require(\"fs\")\|from \"fs\"" ./agents/*Agent.* 2>/dev/null

# Verify FileSystemUtils usage in agent implementations
grep -l 'FileSystemUtils' ./agents/*Agent.* 2>/dev/null | wc -l
```

**Accept when:**
- All agent implementation files in the agents directory import and extend either AgentsMdAgent or implement IAgent interface
- Static analysis confirms no direct filesystem module imports in agent implementations where FileSystemUtils should be used
- All agent implementations pass interface compliance tests demonstrating they implement required base class methods
- New agent implementations follow the established import patterns using relative imports
- Configuration file operations consistently use FileSystemUtils across all agent implementations

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent implementations MUST be checked for base class extension and FileSystemUtils usage before accepting changes to the agents directory.
</enforcement>