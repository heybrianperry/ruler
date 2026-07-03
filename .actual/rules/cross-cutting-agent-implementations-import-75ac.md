# Standardize Core Library Detection and Public API Contracts for Agent System: Agent Implementations Import

These rules are ALWAYS ACTIVE for all agent implementations and core library modules within the system, including all agent implementation files in src/agents/, core utility modules in src/core/, CLI handlers in src/cli/, and configuration loading and processing modules.

### Rules

- **R-AGENT-001** MUST: All agent implementations MUST import core interfaces from './IAgent' or './AbstractAgent' to establish contract compliance.
- **R-AGENT-002** MUST: Core utility modules MUST explicitly export public API contracts as named exports (e.g., loadUnifiedConfig, updateGitignore, discoverSkills, concatenateRules) to establish clear API boundaries.
- **R-AGENT-003** MUST: Configuration parsing functions MUST wrap JSON.parse and parseTOML calls in try-catch blocks with descriptive error messages including file paths and parse positions.
- **R-AGENT-004** MUST: Async file system operations MUST use fs/promises, handle errors explicitly, and document concurrency assumptions following the pattern established by loadUnifiedConfig and discoverSkills.
- **R-AGENT-005** SHOULD: New agent implementations SHOULD extend AbstractAgent or AgentsMdAgent and implement the IAgent interface, importing from './AbstractAgent' and './IAgent' respectively.
- **R-AGENT-006** SHOULD: File system operations SHOULD consistently use Node.js fs, path, or fs/promises modules across all agent implementations and core utilities.

### Verify

```bash
# Verify IAgent/AbstractAgent imports in agent implementations
grep -r "import.*from.*['\"]\./(IAgent|AbstractAgent)['\"]" src/agents/ | wc -l

# Verify public API exports in core utilities
grep -r "export.*{.*}" src/core/*.ts | grep -E "(loadUnifiedConfig|updateGitignore|discoverSkills|concatenateRules)"

# Verify JSON.parse and parseTOML usage with error handling
grep -r "JSON\.parse\|parseTOML" src/core/*.ts src/agents/*.ts

# Verify agent implementations extend base classes
find src/agents -name "*.ts" -exec grep -l "extends.*Agent" {} \;
```

**Accept when:**
- All agent implementation files in src/agents/ import IAgent or AbstractAgent interfaces
- Core utility modules in src/core/ explicitly export public API contracts as named exports
- Configuration parsing operations use JSON.parse or parseTOML with error handling
- File system operations consistently use Node.js fs, path, or fs/promises modules
- All new agent implementations extend AbstractAgent or AgentsMdAgent and implement IAgent interface

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler checks for interface compliance and import resolution are mandatory. Code review checklist verification of IAgent interface implementation for new agents is required. Static analysis tools scanning for direct fs/path imports and public export patterns must pass. CI pipeline grep-based verification commands checking pattern compliance must succeed. Violations block merge and require refactoring or documented exception approval.
</enforcement>