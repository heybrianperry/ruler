# Standardize Core Library Imports for Subagent File Processing: Public Contracts Parsedfrontmatter

These rules are ALWAYS ACTIVE for all modules in src/core/ that process subagent files, specifically SubagentsUtils.ts and SubagentsProcessor.ts, and all functions that parse, validate, discover, or write subagent configuration files.

### Rules

- **R-SUBAGENT-001** SHOULD: Public API contracts (ParsedFrontmatter, CopilotToolMapping) SHOULD be defined in shared type definitions to maintain consistency across all subagent processing modules.
- **R-SUBAGENT-002** MUST: Use 'fs/promises' for all asynchronous file operations in subagent processing modules; do not use synchronous fs methods.
- **R-SUBAGENT-003** MUST: Import 'fs/promises' as a named import (e.g., `import { readFile, writeFile } from 'fs/promises'`) for tree-shaking benefits.
- **R-SUBAGENT-004** MUST: Use 'js-yaml' for YAML parsing and '@iarna/toml' for TOML parsing across all subagent processing modules.
- **R-SUBAGENT-005** MUST: Centralize error handling for YAML/TOML parsing failures in utility functions to provide consistent error messages.
- **R-SUBAGENT-006** MUST: Use path.join() and path.resolve() consistently to avoid hardcoded path separators that break on Windows.
- **R-SUBAGENT-007** SHOULD: Export shared utility functions (parseFrontmatter, validateFrontmatter, loadSubagentFile) from SubagentsUtils.ts and import them in consuming modules.
- **R-SUBAGENT-008** SHOULD: Document the expected structure of ParsedFrontmatter and CopilotToolMapping in TypeScript interfaces with JSDoc comments.

### Verify

```bash
# Verify standard library imports are used
grep -r "import.*from 'fs/promises'" src/core/Subagents*.ts
grep -r "import.*from 'js-yaml'" src/core/Subagents*.ts
grep -r "import.*from '@iarna/toml'" src/core/SubagentsProcessor.ts

# Verify shared utility functions are exported and used
grep -r "parseFrontmatter\|validateFrontmatter\|loadSubagentFile" src/core/SubagentsUtils.ts

# Verify no synchronous fs methods in async contexts
grep -r "fs\.readFileSync\|fs\.writeFileSync" src/core/Subagents*.ts
```

**Accept when:**
- All subagent processing modules import fs/promises, js-yaml, and path from the standard library set
- Shared utility functions (parseFrontmatter, validateFrontmatter, loadSubagentFile) are exported from SubagentsUtils.ts and imported by consuming modules
- No synchronous fs methods are used in SubagentsUtils.ts or SubagentsProcessor.ts
- ParsedFrontmatter and CopilotToolMapping type contracts are consistently used across all subagent processing modules
- All file path operations use path.join() or path.resolve() rather than hardcoded separators

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep commands in CI pipeline MUST detect non-standard library imports. Code review MUST verify library imports in subagent-related PRs. TypeScript compilation MUST ensure shared type contracts are used consistently. Violations result in CI pipeline failure and require architectural approval for exceptions.
</enforcement>