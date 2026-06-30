# Standardize Core Library Imports for Subagent File Processing: Subagent File Processing

These rules are ALWAYS ACTIVE for all subagent file processing modules in src/core/ that parse, validate, discover, or write subagent configuration files.

### Rules

- **R-SUBAGENT-001** MUST: All subagent file processing modules MUST use 'fs/promises' for asynchronous file system operations rather than synchronous fs methods.

### Verify

```bash
# Verify fs/promises imports in subagent processing modules
grep -r "import.*from 'fs/promises'" src/core/Subagents*.ts

# Verify js-yaml imports
grep -r "import.*from 'js-yaml'" src/core/Subagents*.ts

# Verify @iarna/toml imports
grep -r "import.*from '@iarna/toml'" src/core/SubagentsProcessor.ts

# Verify shared utility functions are exported and used
grep -r "parseFrontmatter\|validateFrontmatter\|loadSubagentFile" src/core/SubagentsUtils.ts

# Verify no synchronous fs methods in subagent modules
! grep -r "fs\.readFileSync\|fs\.writeFileSync\|fs\.readdirSync" src/core/Subagents*.ts
```

**Accept when:**
- All subagent processing modules import fs/promises, js-yaml, and path from the standard library set
- Shared utility functions (parseFrontmatter, validateFrontmatter, loadSubagentFile) are exported from SubagentsUtils.ts and imported by consuming modules
- No synchronous fs methods are used in SubagentsUtils.ts or SubagentsProcessor.ts
- Type definitions for ParsedFrontmatter and CopilotToolMapping are consistently used across modules

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep commands and TypeScript compilation checks are mandatory before accepting subagent file processing code.
</enforcement>