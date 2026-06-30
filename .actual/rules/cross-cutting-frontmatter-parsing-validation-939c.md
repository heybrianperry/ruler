# Standardize Core Library Imports for Subagent File Processing: Frontmatter Parsing Validation

These rules are ALWAYS ACTIVE for all modules in src/core/ that process subagent files, specifically SubagentsUtils.ts and SubagentsProcessor.ts, and any functions that parse, validate, discover, or write subagent configuration files.

### Rules

- **R-SUBAGENT-001** MUST: Frontmatter parsing and validation functions (parseFrontmatter, validateFrontmatter) MUST be exported from SubagentsUtils and reused across modules.
- **R-SUBAGENT-002** MUST: All subagent processing modules MUST import fs/promises, js-yaml, and path from the standard library set.
- **R-SUBAGENT-003** MUST: No synchronous fs methods are permitted in SubagentsUtils.ts or SubagentsProcessor.ts.
- **R-SUBAGENT-004** MUST: Shared utility functions (parseFrontmatter, validateFrontmatter, loadSubagentFile) MUST be exported from SubagentsUtils.ts and imported by consuming modules.
- **R-SUBAGENT-005** MUST: Type definitions for subagent data structures (ParsedFrontmatter, CopilotToolMapping) MUST be consistently used across all subagent processing modules.

### Verify

```bash
# Verify standard library imports are used
grep -r "import.*from 'fs/promises'" src/core/Subagents*.ts
grep -r "import.*from 'js-yaml'" src/core/Subagents*.ts
grep -r "import.*from '@iarna/toml'" src/core/SubagentsProcessor.ts

# Verify shared utility functions are exported and used
grep -r "parseFrontmatter\|validateFrontmatter\|loadSubagentFile" src/core/SubagentsUtils.ts

# Verify no synchronous fs methods are used
grep -r "fs\.readFileSync\|fs\.writeFileSync\|fs\.readdirSync" src/core/Subagents*.ts
```

**Accept when:**
- All subagent processing modules import fs/promises, js-yaml, and path from the standard library set
- Shared utility functions (parseFrontmatter, validateFrontmatter, loadSubagentFile) are exported from SubagentsUtils.ts and imported by consuming modules
- No synchronous fs methods are used in SubagentsUtils.ts or SubagentsProcessor.ts
- Type contracts (ParsedFrontmatter, CopilotToolMapping) are consistently applied across all subagent processing modules

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep commands MUST be executed in CI pipeline to detect non-standard library imports. Code review MUST verify library imports in subagent-related PRs. TypeScript compilation MUST check shared type contracts are used consistently. Violations result in CI pipeline failure and require architectural approval for exceptions.
</enforcement>