# Standardize Node.js Core Module Integration for Subagent File System Operations: Yaml Frontmatter Parsing

These rules are ALWAYS ACTIVE for all subagent file system operations and configuration parsing within the core module boundary, specifically targeting SubagentsProcessor.ts and SubagentsUtils.ts.

### Rules

- **R-YAML-001** MUST: YAML frontmatter parsing MUST use the 'js-yaml' library as the standard parser.
- **R-YAML-002** MUST: All file system operations in SubagentsProcessor.ts and SubagentsUtils.ts MUST use fs/promises API exclusively.
- **R-YAML-003** MUST: Path module MUST be imported and used for all path manipulation operations.
- **R-YAML-004** MUST: All file system operations MUST be wrapped in try-catch blocks to handle ENOENT, EACCES, and other common file system errors gracefully.
- **R-YAML-005** MUST: Import 'fs/promises' as a named import to clearly distinguish async operations from legacy fs module usage.
- **R-YAML-006** SHOULD: Use path.join() and path.resolve() consistently to avoid hardcoded path separators that break cross-platform compatibility.
- **R-YAML-007** SHOULD: Document the expected structure of YAML frontmatter and TOML configuration files in schema definitions or JSDoc comments.
- **R-YAML-008** SHOULD: Implement comprehensive validation in validateFrontmatter to catch malformed configurations early in the loading pipeline.

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
grep -E "export.*(discoverSubagents|parseFrontmatter|loadSubagentFile|propagateSubagentsForClaude)" src/core/Subagents*.ts
```

**Accept when:**
- All file system operations in SubagentsProcessor.ts and SubagentsUtils.ts use fs/promises API exclusively
- Path module is imported and used for all path manipulation operations
- js-yaml and @iarna/toml are the only parsing libraries used for structured configuration data
- All public API functions (discoverSubagents, parseFrontmatter, validateFrontmatter, loadSubagentFile, propagateSubagentsForClaude) are exported and documented
- No synchronous fs module operations appear in core subagent modules
- All file system operations are wrapped in try-catch blocks

<enforcement>
Verified by: Static analysis via ESLint rules prohibiting synchronous fs module usage in core subagent modules.
Verified by: Code review checklist requiring verification of fs/promises usage and proper error handling.
Verified by: Integration tests validating public API contracts and concurrency behavior.
Violation handling: CI pipeline fails on detection of synchronous fs operations in SubagentsProcessor or SubagentsUtils.
Violation handling: Pull requests introducing alternative parsing libraries require architecture review and justification.
Violation handling: Breaking changes to public API contracts trigger automated deprecation warnings and version bump requirements.
Claude Code MUST NOT skip or defer verification.
</enforcement>