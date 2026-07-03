# Standardize Core Library Detection and Public API Contracts for Agent System: Async Operations That

These rules are ALWAYS ACTIVE for all agent implementations and core library modules within the system, including files in src/agents/, src/core/, and src/cli/ directories that coordinate agent operations, configuration loading, and file system utilities.

### Rules

- **R-ASYNC-001** SHOULD: Async operations that coordinate file system access SHOULD follow the concurrency model established by functions like loadUnifiedConfig, discoverSkills, and applyRulerConfig.
- **R-ASYNC-002** MUST: New agent implementations MUST extend AbstractAgent or AgentsMdAgent and implement the IAgent interface, importing from './AbstractAgent' and './IAgent' respectively.
- **R-ASYNC-003** MUST: Core utility modules MUST explicitly export public contracts as named exports (e.g., export { loadUnifiedConfig, UnifiedLoadOptions }) to establish clear API boundaries.
- **R-ASYNC-004** MUST: Configuration parsing functions MUST wrap JSON.parse and parseTOML calls in try-catch blocks with descriptive error messages including file paths and parse positions.
- **R-ASYNC-005** MUST: Async file system operations MUST follow the pattern established by loadUnifiedConfig and discoverSkills: use fs/promises, handle errors explicitly, and document concurrency assumptions.
- **R-ASYNC-006** SHOULD: When adding new core libraries, SHOULD update the libs.core.detected pattern documentation and ensure consistent import patterns across related modules.

### Verify

```bash
# Verify IAgent interface imports in agent implementations
grep -r "import.*from.*['\"]\./IAgent['\"]" src/agents/ | wc -l

# Verify public API exports in core utilities
grep -r "export.*{.*}" src/core/*.ts | grep -E "(loadUnifiedConfig|updateGitignore|discoverSkills|concatenateRules)"

# Verify JSON.parse and parseTOML usage with error handling
grep -r "JSON\.parse\|parseTOML" src/core/*.ts src/agents/*.ts

# Verify agent implementations extend AbstractAgent
find src/agents -name "*.ts" -exec grep -l "extends.*Agent" {} \;
```

**Accept when:**
- All agent implementation files in src/agents/ import IAgent or AbstractAgent interfaces
- Core utility modules in src/core/ explicitly export public API contracts as named exports
- Configuration parsing operations use JSON.parse or parseTOML with error handling
- File system operations consistently use Node.js fs, path, or fs/promises modules
- TypeScript compiler successfully validates interface compliance and import resolution
- All async file system coordination functions document their concurrency assumptions

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures block merge for missing interface implementations. Code review feedback requests refactoring to align with established patterns. CI pipeline warnings flag deviations from core library usage patterns. Architecture review required for intentional deviations with documented justification.
</enforcement>