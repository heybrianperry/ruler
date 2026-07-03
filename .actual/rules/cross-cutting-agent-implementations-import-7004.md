# Standardize Core Library Imports for Test Frameworks and File System Operations: Agent Implementations Import

These rules are ALWAYS ACTIVE for all TypeScript files in src/agents/, all test files in tests/ and subdirectories, core utility modules in src/core/, MCP integration modules in src/mcp/, and CLI implementation files requiring file system access.

### Rules

- **R-CORE-001** MUST: Agent implementations MUST import from established utility modules (./IAgent, ../core/FileSystemUtils, ../mcp/merge) rather than duplicating utility functions.
- **R-CORE-002** MUST: Use fs/promises API for all file system operations instead of callback-based or synchronous fs methods, except where documented exceptions apply (EXC-001, EXC-002).
- **R-CORE-003** MUST: All test files MUST use standardized test framework imports (describe, it, expect) from the configured test runner.
- **R-CORE-004** SHOULD: JSON parsing operations SHOULD be wrapped in try-catch blocks or use error handling utilities following file system reads.
- **R-CORE-005** SHOULD: All new agent implementations SHOULD extend AbstractAgent or AgentsMdAgent base classes.

### Verify

```bash
# Check for non-promises fs imports in src/ and tests/
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v 'fs/promises' | wc -l

# Check for non-promises fs imports using ES6 syntax
grep -r "from 'fs'" src/ tests/ --include='*.ts' | grep -v "from 'fs/promises'" | wc -l

# Run ESLint with restricted imports rule
npx eslint src/ tests/ --rule 'no-restricted-imports: [error, {paths: [{name: fs, message: Use fs/promises instead}]}]'

# Check JSON.parse operations for error handling
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