# Standardize Core Library Detection and Public API Contracts for Agent System: Agent Implementations Extend

These rules are ALWAYS ACTIVE for all agent implementations in `src/agents/`, core library modules in `src/core/`, and CLI handlers in `src/cli/` that coordinate agent operations and configuration processing.

### Rules

- **R-AGENT-001** SHOULD: Agent implementations SHOULD extend AgentsMdAgent or AbstractAgent base classes to inherit common configuration application patterns.

### Verify

```bash
# Verify agent implementations import IAgent or AbstractAgent interfaces
grep -r "import.*from.*['\"]\./IAgent['\"]" src/agents/ | wc -l

# Verify core utility modules explicitly export public API contracts
grep -r "export.*{.*}" src/core/*.ts | grep -E "(loadUnifiedConfig|updateGitignore|discoverSkills|concatenateRules)"

# Verify configuration parsing uses JSON.parse or parseTOML with error handling
grep -r "JSON\.parse\|parseTOML" src/core/*.ts src/agents/*.ts

# Verify file system operations use Node.js fs, path, or fs/promises
find src/agents -name "*.ts" -exec grep -l "extends.*Agent" {} \;
```

**Accept when:**
- All agent implementation files in `src/agents/` import IAgent or AbstractAgent interfaces
- Core utility modules in `src/core/` explicitly export public API contracts as named exports
- Configuration parsing operations use JSON.parse or parseTOML with error handling
- File system operations consistently use Node.js fs, path, or fs/promises modules
- New agent implementations extend AbstractAgent or AgentsMdAgent and implement the IAgent interface
- Core utility modules establish clear API boundaries through explicit named exports

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. TypeScript compiler checks, code review checklists, static analysis tools, and CI pipeline verification commands are mandatory before accepting changes to agent implementations or core library modules.
</enforcement>