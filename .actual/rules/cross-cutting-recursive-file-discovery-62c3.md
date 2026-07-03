# Standardize Node.js Core Module Integration for Subagent File System Operations: Recursive File Discovery

These rules are ALWAYS ACTIVE for all subagent file system operations and configuration parsing within the core module boundary, specifically targeting SubagentsProcessor.ts and SubagentsUtils.ts.

### Rules

- **R-SUBAGENT-001** SHOULD: Recursive file discovery operations SHOULD be encapsulated in dedicated functions (listMarkdownFilesRecursive) to maintain separation of concerns.
- **R-SUBAGENT-002** MUST: All file system operations in SubagentsProcessor.ts and SubagentsUtils.ts MUST use fs/promises API exclusively (no synchronous fs module usage).
- **R-SUBAGENT-003** MUST: Path module MUST be imported and used for all path manipulation operations to ensure cross-platform compatibility.
- **R-SUBAGENT-004** MUST: js-yaml and @iarna/toml MUST be the only parsing libraries used for structured configuration data.
- **R-SUBAGENT-005** MUST: All file system operations MUST be wrapped in try-catch blocks to handle ENOENT, EACCES, and other common file system errors gracefully.
- **R-SUBAGENT-006** MUST: All public API functions (discoverSubagents, parseFrontmatter, validateFrontmatter, loadSubagentFile, propagateSubagentsForClaude) MUST be exported and documented.
- **R-SUBAGENT-007** SHOULD: Atomic write operations (writeAgentsDirectoryAtomic) and Set-based deduplication SHOULD be used to ensure consistency during concurrent subagent discovery.

### Verify

```bash
# Verify no synchronous fs module usage in core subagent modules
grep -r "require('fs')" src/core/Subagents*.ts | grep -v fs/promises

# Verify path module is imported
grep -r "from 'path'" src/core/Subagents*.ts

# Verify js-yaml is imported
grep -r "from 'js-yaml'" src/core/Subagents*.ts

# Verify @iarna/toml is imported
grep -r "from '@iarna/toml'" src/core/Subagents*.ts

# Verify public API functions are exported
grep -E "export.*(discoverSubagents|parseFrontmatter|loadSubagentFile|propagateSubagentsForClaude)" src/core/Subagents*.ts
```

**Accept when:**
- All file system operations in SubagentsProcessor.ts and SubagentsUtils.ts use fs/promises API exclusively
- Path module is imported and used for all path manipulation operations
- js-yaml and @iarna/toml are the only parsing libraries used for structured configuration data
- All public API functions (discoverSubagents, parseFrontmatter, validateFrontmatter, loadSubagentFile, propagateSubagentsForClaude) are exported and documented
- Recursive file discovery is encapsulated in dedicated functions like listMarkdownFilesRecursive
- All file system operations include proper error handling for common file system errors

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail on detection of synchronous fs operations in SubagentsProcessor or SubagentsUtils. Pull requests introducing alternative parsing libraries MUST require architecture review. Breaking changes to public API contracts MUST trigger automated deprecation warnings and version bump requirements.
</enforcement>