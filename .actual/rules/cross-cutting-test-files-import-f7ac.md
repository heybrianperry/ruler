# Standardize Core Library Imports for Test Frameworks and File System Operations: Test Files Import

These rules are ALWAYS ACTIVE for all TypeScript files in src/agents/, tests/, src/core/, src/mcp/, and CLI implementation files requiring file system access.

### Rules

- **R-TEST-001** MUST: Test files MUST import test framework primitives (describe, it, expect, beforeEach, afterEach, beforeAll, afterAll) from the configured test framework.
- **R-TEST-002** MUST: All file system operations MUST use fs/promises API with async/await patterns rather than callback-based or synchronous fs methods.
- **R-TEST-003** MUST: JSON.parse operations MUST be wrapped in try-catch blocks or use error handling utilities.
- **R-TEST-004** SHOULD: Agent implementations SHOULD extend AbstractAgent or AgentsMdAgent base classes and import core utility modules (FileSystemUtils, IAgent interface).
- **R-TEST-005** SHOULD: Path operations SHOULD use the path module for cross-platform compatibility.

### Verify

```bash
# Check for non-promises fs imports
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v 'fs/promises' | wc -l

# Check for non-promises fs imports using ES6 syntax
grep -r "from 'fs'" src/ tests/ --include='*.ts' | grep -v "from 'fs/promises'" | wc -l

# Run ESLint with restricted imports rule
npx eslint src/ tests/ --rule 'no-restricted-imports: [error, {paths: [{name: fs, message: Use fs/promises instead}]}]'

# Check JSON.parse error handling coverage
grep -r "JSON.parse" src/ tests/ --include='*.ts' -A 2 -B 2 | grep -c 'try\|catch'
```

**Accept when:**
- All grep commands for non-promises fs imports return zero matches in src/ and tests/ directories
- ESLint validation passes with no restricted import violations for fs module
- At least 90% of JSON.parse operations are wrapped in try-catch blocks or use error handling utilities
- All new agent implementations extend AbstractAgent or AgentsMdAgent base classes
- All test files use standardized test framework imports (describe, it, expect) from the configured test runner

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All violations MUST be flagged during code review and CI pipeline execution.
</enforcement>