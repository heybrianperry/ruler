# Standardize Node.js Core Module Integration for Subagent File System Operations: Subagent Target Collection

These rules are ALWAYS ACTIVE for all subagent file system operations and configuration parsing within the core module boundary, specifically targeting SubagentsProcessor.ts and SubagentsUtils.ts.

### Rules

- **R-SUBAGENT-001** MUST: Subagent target collection operations MUST use Set data structures (targets.add, mapped.add) to ensure uniqueness.

### Verify

```bash
# Verify no synchronous fs module usage in core subagent modules
grep -r "require('fs')" src/core/Subagents*.ts | grep -v fs/promises

# Verify path module is imported and used
grep -r "from 'path'" src/core/Subagents*.ts

# Verify js-yaml is imported for YAML parsing
grep -r "from 'js-yaml'" src/core/Subagents*.ts

# Verify @iarna/toml is imported for TOML parsing
grep -r "from '@iarna/toml'" src/core/Subagents*.ts

# Verify public API functions are exported
grep -E "export.*(discoverSubagents|parseFrontmatter|loadSubagentFile|propagateSubagentsForClaude)" src/core/Subagents*.ts
```

**Accept when:**
- All file system operations in SubagentsProcessor.ts and SubagentsUtils.ts use fs/promises API exclusively
- Path module is imported and used for all path manipulation operations
- js-yaml and @iarna/toml are the only parsing libraries used for structured configuration data
- All public API functions (discoverSubagents, parseFrontmatter, validateFrontmatter, loadSubagentFile, propagateSubagentsForClaude) are exported and documented
- Set data structures are used consistently for target collection operations (targets.add, mapped.add)

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint rules prohibiting synchronous fs module usage in core subagent modules is mandatory. Code review checklist must verify fs/promises usage and proper error handling. CI pipeline MUST fail on detection of synchronous fs operations in SubagentsProcessor or SubagentsUtils.
</enforcement>