# Standardize Core Library Imports for Subagent File Processing: Subagent Discovery Functions

These rules are ALWAYS ACTIVE for all modules in src/core/ that process subagent files, specifically SubagentsUtils.ts and SubagentsProcessor.ts, and all functions that parse, validate, discover, or write subagent configuration files.

### Rules

- **R-SUBAGENT-001** SHOULD: Subagent discovery functions (discoverSubagents, listMarkdownFilesRecursive) SHOULD be centralized in SubagentsProcessor to avoid duplication.
- **R-SUBAGENT-002** MUST: All subagent processing modules MUST import fs/promises as named imports (e.g., `import { readFile, writeFile } from 'fs/promises'`) for tree-shaking benefits.
- **R-SUBAGENT-003** MUST: All subagent processing modules MUST use js-yaml for YAML parsing and @iarna/toml for TOML parsing to ensure consistent behavior.
- **R-SUBAGENT-004** MUST: Shared utility functions (parseFrontmatter, validateFrontmatter, loadSubagentFile) MUST be exported from SubagentsUtils.ts and imported by consuming modules.
- **R-SUBAGENT-005** MUST: No synchronous fs methods (readFileSync, writeFileSync, etc.) MUST be used in SubagentsUtils.ts or SubagentsProcessor.ts.
- **R-SUBAGENT-006** MUST: Error handling for YAML/TOML parsing failures MUST be centralized in utility functions to provide consistent error messages.
- **R-SUBAGENT-007** MUST: Path operations MUST use path.join() and path.resolve() consistently to avoid hardcoded path separators that break on Windows.
- **R-SUBAGENT-008** MUST: Type definitions for subagent data structures (ParsedFrontmatter, CopilotToolMapping) MUST be documented in TypeScript interfaces with JSDoc comments.

### Verify

```bash
# Verify fs/promises imports are used
grep -r "import.*from 'fs/promises'" src/core/Subagents*.ts

# Verify js-yaml imports are used
grep -r "import.*from 'js-yaml'" src/core/Subagents*.ts

# Verify @iarna/toml imports are used
grep -r "import.*from '@iarna/toml'" src/core/SubagentsProcessor.ts

# Verify shared utility functions are exported and used
grep -r "parseFrontmatter\|validateFrontmatter\|loadSubagentFile" src/core/SubagentsUtils.ts

# Verify no synchronous fs methods are used
grep -r "readFileSync\|writeFileSync\|readdirSync" src/core/Subagents*.ts
```

**Accept when:**
- All subagent processing modules import fs/promises, js-yaml, and path from the standard library set
- Shared utility functions (parseFrontmatter, validateFrontmatter, loadSubagentFile) are exported from SubagentsUtils.ts and imported by consuming modules
- No synchronous fs methods are used in SubagentsUtils.ts or SubagentsProcessor.ts
- All path operations use path.join() or path.resolve()
- Type definitions include JSDoc comments documenting expected structure

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep commands MUST be run in CI pipeline to detect non-standard library imports. Code review MUST verify library imports in subagent-related PRs. TypeScript compilation MUST check shared type contracts are used consistently. Violations result in CI pipeline failure and require architectural approval for exceptions.
</enforcement>