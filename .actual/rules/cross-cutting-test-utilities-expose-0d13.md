# Standardize Node.js Core Module Integration for Subagent File System Operations: Test Utilities Expose

These rules are ALWAYS ACTIVE for all subagent file system operations and configuration parsing within the core module boundary, specifically for test utilities that expose internal reset functions.

### Rules

- **R-TEST-001** MAY: Test utilities MAY expose internal reset functions (_resetExperimentalWarningForTests) for testing purposes only.

### Verify

```bash
# Verify test utilities expose reset functions appropriately
grep -r "_resetExperimentalWarningForTests" src/ tests/

# Verify fs/promises usage in core subagent modules
grep -r "require('fs')" src/core/Subagents*.ts | grep -v fs/promises

# Verify path module imports
grep -r "from 'path'" src/core/Subagents*.ts

# Verify js-yaml imports
grep -r "from 'js-yaml'" src/core/Subagents*.ts

# Verify @iarna/toml imports
grep -r "from '@iarna/toml'" src/core/Subagents*.ts

# Verify public API exports
grep -E "export.*(discoverSubagents|parseFrontmatter|loadSubagentFile|propagateSubagentsForClaude)" src/core/Subagents*.ts
```

**Accept when:**
- Test utilities that expose _resetExperimentalWarningForTests are clearly marked as test-only and documented
- All file system operations in SubagentsProcessor.ts and SubagentsUtils.ts use fs/promises API exclusively
- Path module is imported and used for all path manipulation operations
- js-yaml and @iarna/toml are the only parsing libraries used for structured configuration data
- All public API functions (discoverSubagents, parseFrontmatter, validateFrontmatter, loadSubagentFile, propagateSubagentsForClaude) are exported and documented
- Reset functions are only accessible in test environments and not exposed in production builds

<enforcement>
Claude Code MUST NOT skip or defer verification of test utility exposure rules. All internal reset functions must be explicitly scoped to test-only contexts and verified before acceptance.
</enforcement>