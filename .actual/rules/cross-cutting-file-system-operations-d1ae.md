# Standardize Core Library Imports for Test Frameworks and File System Operations: File System Operations

These rules are ALWAYS ACTIVE for all TypeScript files in src/agents/, tests/, src/core/, src/mcp/, and CLI implementation files requiring file system access.

### Rules

- **R-FS-001** MUST: All file system operations MUST use the fs/promises API (import fs from 'fs/promises') rather than callback-based or synchronous fs methods.

### Verify

```bash
# Check for non-promises fs requires
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v 'fs/promises' | wc -l

# Check for non-promises fs imports
grep -r "from 'fs'" src/ tests/ --include='*.ts' | grep -v "from 'fs/promises'" | wc -l

# Run ESLint with restricted imports rule
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
Claude Code MUST NOT skip or defer verification of fs/promises compliance. All file system operations must be audited before accepting changes.
</enforcement>