# Standardize Core Library Imports for Subagent File Processing: Modules Extend Core

These rules are ALWAYS ACTIVE for all modules in src/core/ that process subagent files, specifically SubagentsUtils.ts and SubagentsProcessor.ts, and all functions that parse, validate, discover, or write subagent configuration files.

### Rules

- **R-CORE-001** MUST: Import `fs/promises` as a named import (e.g., `import { readFile, writeFile } from 'fs/promises'`) for tree-shaking benefits in all subagent file processing modules.
- **R-CORE-002** MUST: Use `js-yaml` for YAML parsing across all subagent processing modules that handle YAML configuration files.
- **R-CORE-003** MUST: Use `@iarna/toml` for TOML parsing in SubagentsProcessor.ts and related modules that handle TOML configuration files.
- **R-CORE-004** MUST: Import `path` module and use `path.join()` and `path.resolve()` consistently to avoid hardcoded path separators that break on Windows.
- **R-CORE-005** MUST: Centralize error handling for YAML/TOML parsing failures in utility functions to provide consistent error messages.
- **R-CORE-006** MUST: Export shared utility functions (parseFrontmatter, validateFrontmatter, loadSubagentFile) from SubagentsUtils.ts and import them in consuming modules.
- **R-CORE-007** MUST: Use shared API contracts (ParsedFrontmatter, CopilotToolMapping) consistently across all subagent processing modules.
- **R-CORE-008** MUST NOT: Use synchronous fs methods in SubagentsUtils.ts or SubagentsProcessor.ts; all file operations must use fs/promises for non-blocking I/O.
- **R-CORE-009** MAY: Modules MAY extend the core library set with additional parsing libraries if new configuration formats are introduced, subject to architectural review.
- **R-CORE-010** SHOULD: Document the expected structure of ParsedFrontmatter and CopilotToolMapping in TypeScript interfaces with JSDoc comments.

### Verify

```bash
# Verify fs/promises imports in subagent processing modules
grep -r "import.*from 'fs/promises'" src/core/Subagents*.ts

# Verify js-yaml imports in subagent processing modules
grep -r "import.*from 'js-yaml'" src/core/Subagents*.ts

# Verify @iarna/toml imports in SubagentsProcessor
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
- ParsedFrontmatter and CopilotToolMapping type contracts are consistently used across modules
- path.join() and path.resolve() are used for all path operations

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep commands in CI pipeline must detect non-standard library imports. Code review must verify library imports in subagent-related PRs. TypeScript compilation must ensure shared type contracts are used consistently. Violations result in CI pipeline failure and require architectural approval for exceptions.
</enforcement>