# Standardize Core Library Imports for Test Frameworks and File System Operations: Test Harness Utilities

These rules are ALWAYS ACTIVE for all TypeScript files in src/agents/, tests/, src/core/, src/mcp/, and CLI implementation files requiring file system access.

### Rules

- **R-HARNESS-001** SHOULD: Test harness utilities SHOULD be imported from centralized test utility modules (./harness) to promote code reuse.
- **R-HARNESS-002** MUST: Use fs/promises API for all file system operations in async contexts to ensure consistent async/await patterns.
- **R-HARNESS-003** MUST: Import describe/it/expect from test frameworks in all test files to maintain standardized test structure.
- **R-HARNESS-004** SHOULD: Centralize utility imports (FileSystemUtils, IAgent interface) to reduce code duplication across agent implementations.
- **R-HARNESS-005** MUST: Wrap JSON.parse operations in try-catch blocks or use error handling utilities for consistent error handling.

### Verify

```bash
# Check for non-promises fs imports
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v 'fs/promises' | wc -l

# Check for callback-based fs imports
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
Claude Code MUST NOT skip or defer verification of these rules. All verification commands MUST pass before accepting changes to files in scope.
</enforcement>