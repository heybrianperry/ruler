# Standardize Core Library Detection and Public API Contracts for Agent System: Modules Use Additional

These rules are ALWAYS ACTIVE for all agent implementations and core library modules within the system, including files in src/agents/, src/core/, and src/cli/ directories.

### Rules

- **R-CORE-001** MAY: Modules MAY use additional libraries (e.g., @iarna/toml for TOML parsing) when required for specific format support, provided they expose consistent public contracts.
- **R-CORE-002** MUST: New agent implementations should extend AbstractAgent or AgentsMdAgent and implement the IAgent interface, importing from './AbstractAgent' and './IAgent' respectively.
- **R-CORE-003** MUST: Core utility modules should explicitly export public contracts as named exports (e.g., export { loadUnifiedConfig, UnifiedLoadOptions }) to establish clear API boundaries.
- **R-CORE-004** MUST: Configuration parsing functions should wrap JSON.parse and parseTOML calls in try-catch blocks with descriptive error messages including file paths and parse positions.
- **R-CORE-005** MUST: Async file system operations should follow the pattern established by loadUnifiedConfig and discoverSkills: use fs/promises, handle errors explicitly, and document concurrency assumptions.
- **R-CORE-006** SHOULD: When adding new core libraries, update the libs.core.detected pattern documentation and ensure consistent import patterns across related modules.

### Verify

```bash
# Verify IAgent interface imports in agent implementations
grep -r "import.*from.*['\"]\./IAgent['\"]" src/agents/ | wc -l

# Verify public API contract exports in core modules
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

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures block merge for missing interface implementations. Code review feedback requests refactoring to align with established patterns. CI pipeline warnings flag deviations from core library usage patterns. Architecture review required for intentional deviations with documented justification.
</enforcement>