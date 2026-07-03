# Standardize Core Library Detection and Public API Contracts for Agent System: Configuration Parsing Validate

These rules are ALWAYS ACTIVE for all agent implementations and core library modules within the system, including files in src/agents/, src/core/, and src/cli/ directories that handle configuration parsing, file system operations, and public API contracts.

### Rules

- **R-CONFIG-001** MUST: Configuration parsing MUST validate input through JSON.parse or parseTOML operations before processing.

### Verify

```bash
# Verify agent implementations import IAgent or AbstractAgent interfaces
grep -r "import.*from.*['\"]\./(IAgent|AbstractAgent)['\"]" src/agents/ | wc -l

# Verify core utility modules export public API contracts
grep -r "export.*{.*}" src/core/*.ts | grep -E "(loadUnifiedConfig|updateGitignore|discoverSkills|concatenateRules)"

# Verify configuration parsing uses JSON.parse or parseTOML with error handling
grep -r "JSON\.parse\|parseTOML" src/core/*.ts src/agents/*.ts

# Verify file system operations use Node.js fs, path, or fs/promises
find src/agents -name "*.ts" -exec grep -l "import.*from.*['\"]fs" {} \;
find src/core -name "*.ts" -exec grep -l "import.*from.*['\"]fs" {} \;
```

**Accept when:**
- All agent implementation files in src/agents/ import IAgent or AbstractAgent interfaces
- Core utility modules in src/core/ explicitly export public API contracts as named exports
- Configuration parsing operations use JSON.parse or parseTOML with error handling (try-catch blocks)
- File system operations consistently use Node.js fs, path, or fs/promises modules
- New agent implementations extend AbstractAgent or AgentsMdAgent and implement IAgent interface
- Configuration parsing functions wrap JSON.parse and parseTOML calls with descriptive error messages

<enforcement>
Clause MUST NOT skip or defer verification. TypeScript compiler checks, code review checklists, static analysis tools, and CI pipeline verification commands are mandatory before merge.
</enforcement>