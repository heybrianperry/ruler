# Standardize Core Library Imports for Test Frameworks and File System Operations: Path Manipulation Operations

These rules are ALWAYS ACTIVE for all TypeScript files in src/agents/, all test files in tests/ and subdirectories, core utility modules in src/core/, MCP integration modules in src/mcp/, and CLI implementation files requiring file system access.

### Rules

- **R-PATH-001** MUST: Path manipulation operations MUST use the Node.js 'path' module to ensure cross-platform compatibility.

### Verify

```bash
# Check for non-promises fs imports
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v 'fs/promises' | wc -l

# Check for non-promises fs imports using ES6 syntax
grep -r "from 'fs'" src/ tests/ --include='*.ts' | grep -v "from 'fs/promises'" | wc -l

# Verify ESLint rules for restricted fs imports
npx eslint src/ tests/ --rule 'no-restricted-imports: [error, {paths: [{name: fs, message: Use fs/promises instead}]}]'

# Verify JSON.parse operations have error handling
grep -r "JSON.parse" src/ tests/ --include='*.ts' -A 2 -B 2 | grep -c 'try\|catch'
```

**Accept when:**
- All grep commands for non-promises fs imports return zero matches in src/ and tests/ directories
- ESLint validation passes with no restricted import violations for fs module
- At least 90% of JSON.parse operations are wrapped in try-catch blocks or use error handling utilities
- All new agent implementations extend AbstractAgent or AgentsMdAgent base classes
- All test files use standardized test framework imports (describe, it, expect) from the configured test runner

<enforcement>
Claude Code MUST NOT skip or defer verification of path module usage and fs/promises compliance. All violations must be remediated before code review approval.
</enforcement>