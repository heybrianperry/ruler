# Standardize Core Library Imports for Test Frameworks and File System Operations: Json Parsing Operations

These rules are ALWAYS ACTIVE for all TypeScript files in src/agents/, tests/, src/core/, and src/mcp/ directories that perform JSON parsing operations following file system reads.

### Rules

- **R-JSON-001** MUST: JSON parsing operations following file reads MUST include error handling for malformed JSON content.

### Verify

```bash
# Check for JSON.parse operations without try-catch or error handling
grep -r "JSON.parse" src/ tests/ --include='*.ts' -A 2 -B 2 | grep -c 'try\|catch'

# Verify fs/promises imports are used (not callback-based fs)
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v 'fs/promises' | wc -l
grep -r "from 'fs'" src/ tests/ --include='*.ts' | grep -v "from 'fs/promises'" | wc -l

# Run ESLint to catch restricted fs imports
npx eslint src/ tests/ --rule 'no-restricted-imports: [error, {paths: [{name: fs, message: Use fs/promises instead}]}]'
```

**Accept when:**
- At least 90% of JSON.parse operations are wrapped in try-catch blocks or use error handling utilities
- All grep commands for non-promises fs imports return zero matches in src/ and tests/ directories
- ESLint validation passes with no restricted import violations for fs module
- All new agent implementations extend AbstractAgent or AgentsMdAgent base classes
- All test files use standardized test framework imports (describe, it, expect) from the configured test runner

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON parsing operations must be audited for error handling compliance before code review approval.
</enforcement>