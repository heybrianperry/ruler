# Standardize Node.js Core Module Integration for Subagent File System Operations: Public Contracts Discoversubagents

These rules are ALWAYS ACTIVE for all subagent file system operations and configuration parsing within the core module boundary, specifically targeting SubagentsProcessor.ts and SubagentsUtils.ts.

### Rules

- **R-SUBAGENTS-001** MUST: Public API contracts (discoverSubagents, getSelectedSubagentTargets, getSubagentsGitignorePaths, propagateSubagentsForClaude, parseFrontmatter, validateFrontmatter, loadSubagentFile) MUST be exported and maintained for external consumers.
- **R-SUBAGENTS-002** MUST: All file system operations in SubagentsProcessor.ts and SubagentsUtils.ts MUST use fs/promises API exclusively; synchronous fs module usage is prohibited.
- **R-SUBAGENTS-003** MUST: Path module MUST be imported and used for all path manipulation operations to ensure cross-platform compatibility.
- **R-SUBAGENTS-004** MUST: js-yaml and @iarna/toml MUST be the only parsing libraries used for structured configuration data; alternative parsers require architecture review.
- **R-SUBAGENTS-005** MUST: All file system operations MUST be wrapped in try-catch blocks to handle ENOENT, EACCES, and other common file system errors gracefully.
- **R-SUBAGENTS-006** SHOULD: Use atomic write operations (writeAgentsDirectoryAtomic) and Set-based deduplication to ensure consistency during concurrent subagent discovery.
- **R-SUBAGENTS-007** SHOULD: Document the expected structure of YAML frontmatter and TOML configuration files in schema definitions or JSDoc comments.

### Verify

```bash
# Verify no synchronous fs operations in core subagent modules
grep -r "require('fs')" src/core/Subagents*.ts | grep -v fs/promises

# Verify path module is imported
grep -r "from 'path'" src/core/Subagents*.ts

# Verify js-yaml is imported
grep -r "from 'js-yaml'" src/core/Subagents*.ts

# Verify @iarna/toml is imported
grep -r "from '@iarna/toml'" src/core/Subagents*.ts

# Verify public API functions are exported
grep -E "export.*(discoverSubagents|parseFrontmatter|loadSubagentFile|propagateSubagentsForClaude|getSelectedSubagentTargets|getSubagentsGitignorePaths|validateFrontmatter)" src/core/Subagents*.ts
```

**Accept when:**
- All file system operations in SubagentsProcessor.ts and SubagentsUtils.ts use fs/promises API exclusively
- Path module is imported and used for all path manipulation operations
- js-yaml and @iarna/toml are the only parsing libraries used for structured configuration data
- All public API functions (discoverSubagents, parseFrontmatter, validateFrontmatter, loadSubagentFile, propagateSubagentsForClaude, getSelectedSubagentTargets, getSubagentsGitignorePaths) are exported and documented
- All file system operations are wrapped in try-catch blocks with appropriate error handling

<enforcement>
Verified by: Static analysis via ESLint rules prohibiting synchronous fs module usage in core subagent modules. Verified by: Code review checklist requiring verification of fs/promises usage and proper error handling. Verified by: Integration tests validating public API contracts and concurrency behavior. Claude Code MUST NOT skip or defer verification. Violation handling: CI pipeline fails on detection of synchronous fs operations in SubagentsProcessor or SubagentsUtils. Violation handling: Pull requests introducing alternative parsing libraries require architecture review and justification. Violation handling: Breaking changes to public API contracts trigger automated deprecation warnings and version bump requirements.
</enforcement>