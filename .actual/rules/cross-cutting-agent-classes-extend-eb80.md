# Standardize Core Library Imports for Test Frameworks and File System Operations: Agent Classes Extend

These rules are ALWAYS ACTIVE for all TypeScript files in src/agents/, all test files in tests/ and subdirectories, core utility modules in src/core/, MCP integration modules in src/mcp/, and CLI implementation files requiring file system access.

### Rules

- **R-CORE-001** MUST: Agent classes MUST extend base classes (AbstractAgent, AgentsMdAgent) to inherit common functionality rather than reimplementing patterns.
- **R-CORE-002** SHOULD: All file system operations SHOULD use fs/promises API with async/await patterns rather than callback-based or synchronous fs methods.
- **R-CORE-003** SHOULD: Test files SHOULD use standardized test framework imports (describe, it, expect) alongside fs/promises and path modules.
- **R-CORE-004** SHOULD: JSON parsing operations SHOULD follow file system reads and be wrapped in try-catch blocks or use error handling utilities.
- **R-CORE-005** SHOULD: Path manipulations SHOULD use the path module consistently for cross-platform compatibility.

### Verify

```bash
# Check for non-promises fs imports in src/ and tests/
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v 'fs/promises' | wc -l

# Check for non-promises fs imports using ES6 syntax
grep -r "from 'fs'" src/ tests/ --include='*.ts' | grep -v "from 'fs/promises'" | wc -l

# Run ESLint with restricted imports rule
npx eslint src/ tests/ --rule 'no-restricted-imports: [error, {paths: [{name: fs, message: Use fs/promises instead}]}]'

# Verify JSON.parse operations have error handling
grep -r "JSON.parse" src/ tests/ --include='*.ts' -A 2 -B 2 | grep -c 'try\|catch'

# Verify agent classes extend base classes
grep -r "class.*Agent.*extends" src/agents/ --include='*.ts' | wc -l
```

**Accept when:**
- All grep commands for non-promises fs imports return zero matches in src/ and tests/ directories
- ESLint validation passes with no restricted import violations for fs module
- At least 90% of JSON.parse operations are wrapped in try-catch blocks or use error handling utilities
- All new agent implementations extend AbstractAgent or AgentsMdAgent base classes
- All test files use standardized test framework imports (describe, it, expect) from the configured test runner

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. ESLint violations and non-compliant imports MUST be resolved before code review approval.
</enforcement>