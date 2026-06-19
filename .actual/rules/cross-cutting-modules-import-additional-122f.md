# Standardize Core Library Imports for Test Frameworks and File System Operations: Modules Import Additional

These rules are ALWAYS ACTIVE for all TypeScript files in src/agents/, all test files in tests/ and subdirectories, core utility modules in src/core/, MCP integration modules in src/mcp/, and CLI implementation files requiring file system access.

### Rules

- **R-CORE-001** MUST: Use `fs/promises` for all file system operations in async contexts rather than callback-based or synchronous fs methods.
- **R-CORE-002** MUST: Import test framework functions (describe, it, expect) from the configured test runner in all test files.
- **R-CORE-003** MAY: Modules MAY import additional Node.js core libraries (os, child_process) when required for specific functionality.
- **R-CORE-004** SHOULD: Wrap JSON.parse operations in try-catch blocks or use error handling utilities to ensure consistent error handling.
- **R-CORE-005** SHOULD: Use the path module for all file path operations to ensure cross-platform compatibility.
- **R-CORE-006** MUST: Agent implementations MUST extend AbstractAgent or AgentsMdAgent base classes and import centralized utility modules (FileSystemUtils, IAgent interface).

### Verify

```bash
# Check for non-promises fs imports
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v 'fs/promises' | wc -l

# Check for non-promises fs imports using ES6 syntax
grep -r "from 'fs'" src/ tests/ --include='*.ts' | grep -v "from 'fs/promises'" | wc -l

# Run ESLint with restricted imports rule
npx eslint src/ tests/ --rule 'no-restricted-imports: [error, {paths: [{name: fs, message: Use fs/promises instead}]}]'

# Verify JSON.parse error handling
grep -r "JSON.parse" src/ tests/ --include='*.ts' -A 2 -B 2 | grep -c 'try\|catch'
```

**Accept when:**
- All grep commands for non-promises fs imports return zero matches in src/ and tests/ directories
- ESLint validation passes with no restricted import violations for fs module
- At least 90% of JSON.parse operations are wrapped in try-catch blocks or use error handling utilities
- All new agent implementations extend AbstractAgent or AgentsMdAgent base classes
- All test files use standardized test framework imports (describe, it, expect) from the configured test runner

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules must be verified before accepting code changes in affected scopes.
</enforcement>