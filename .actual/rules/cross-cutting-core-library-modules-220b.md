# Standardize Core Library Detection and Public API Contracts for Agent System: Core Library Modules

These rules are ALWAYS ACTIVE for all agent implementations and core library modules within the system, including all files in src/agents/, src/core/, and src/cli/ directories.

### Rules

- **R-CORE-001** MUST: Core library modules MUST explicitly declare public API contracts through exported types and functions (e.g., IAgent, IAgentConfig, loadUnifiedConfig, updateGitignore).
- **R-CORE-002** MUST: All agent implementation files in src/agents/ MUST import IAgent or AbstractAgent interfaces from established contract modules.
- **R-CORE-003** MUST: Core utility modules in src/core/ MUST explicitly export public API contracts as named exports to establish clear API boundaries.
- **R-CORE-004** MUST: Configuration parsing operations MUST use JSON.parse or parseTOML with error handling wrapped in try-catch blocks with descriptive error messages including file paths.
- **R-CORE-005** MUST: File system operations MUST consistently use Node.js fs, path, or fs/promises modules following established async patterns.
- **R-CORE-006** SHOULD: New agent implementations SHOULD extend AbstractAgent or AgentsMdAgent and implement the IAgent interface.
- **R-CORE-007** SHOULD: Async file system operations SHOULD follow the pattern established by loadUnifiedConfig and discoverSkills: use fs/promises, handle errors explicitly, and document concurrency assumptions.

### Verify

```bash
# Verify IAgent interface imports in agent implementations
grep -r "import.*from.*['\"]\./(IAgent|AbstractAgent)['\"]" src/agents/ | wc -l

# Verify public API exports in core modules
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
- TypeScript compiler successfully resolves all interface imports and type checks pass

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures block merge for missing interface implementations. Code review feedback requests refactoring to align with established patterns. CI pipeline warnings flag deviations from core library usage patterns.
</enforcement>