# Standardize Core Library Detection and Public API Contracts for Agent System: File System Operations

These rules are ALWAYS ACTIVE for all agent implementations and core library modules within the system, including files in src/agents/, src/core/, and src/cli/ directories that perform file system operations or define public API contracts.

### Rules

- **R-FSOPS-001** MUST: File system operations MUST use Node.js 'fs', 'path', or 'fs/promises' modules as detected core libraries.
- **R-FSOPS-002** MUST: New agent implementations MUST extend AbstractAgent or AgentsMdAgent and implement the IAgent interface.
- **R-FSOPS-003** MUST: Core utility modules MUST explicitly export public contracts as named exports to establish clear API boundaries.
- **R-FSOPS-004** MUST: Configuration parsing functions MUST wrap JSON.parse and parseTOML calls in try-catch blocks with descriptive error messages including file paths.
- **R-FSOPS-005** MUST: Async file system operations MUST use fs/promises, handle errors explicitly, and document concurrency assumptions.
- **R-FSOPS-006** SHOULD: When adding new core libraries, update the libs.core.detected pattern documentation and ensure consistent import patterns across related modules.

### Verify

```bash
# Verify IAgent interface imports in agent implementations
grep -r "import.*from.*['\"]\./(IAgent|AbstractAgent)['\"]" src/agents/ | wc -l

# Verify public API exports in core utilities
grep -r "export.*{.*}" src/core/*.ts | grep -E "(loadUnifiedConfig|updateGitignore|discoverSkills|concatenateRules)"

# Verify configuration parsing with error handling
grep -r "JSON\.parse\|parseTOML" src/core/*.ts src/agents/*.ts

# Verify agent implementations extend base classes
find src/agents -name "*.ts" -exec grep -l "extends.*Agent" {} \;

# Verify fs/path/fs/promises imports
grep -r "from.*['\"]fs['\"]\|from.*['\"]path['\"]\|from.*['\"]fs/promises['\"]" src/agents/ src/core/ src/cli/
```

**Accept when:**
- All agent implementation files in src/agents/ import IAgent or AbstractAgent interfaces
- Core utility modules in src/core/ explicitly export public API contracts as named exports
- Configuration parsing operations use JSON.parse or parseTOML with error handling in try-catch blocks
- File system operations consistently use Node.js fs, path, or fs/promises modules
- All new agent implementations extend AbstractAgent or AgentsMdAgent and implement IAgent
- Async file system operations follow established patterns with explicit error handling

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler checks, code review checklists, static analysis tools, and CI pipeline verification commands are mandatory before accepting changes to agent implementations or core library modules.
</enforcement>