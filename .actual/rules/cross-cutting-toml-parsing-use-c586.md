# Standardize Core Library Imports for Subagent File Processing: Toml Parsing Use

These rules are ALWAYS ACTIVE for all modules in src/core/ that process subagent files, specifically SubagentsUtils.ts and SubagentsProcessor.ts, and all functions that parse, validate, discover, or write subagent configuration files.

### Rules

- **R-TOML-001** MUST: TOML parsing MUST use the '@iarna/toml' library when processing TOML-formatted subagent configurations.
- **R-TOML-002** MUST: Import 'fs/promises' as a named import (e.g., `import { readFile, writeFile } from 'fs/promises'`) for tree-shaking benefits.
- **R-TOML-003** MUST: Use path.join() and path.resolve() consistently to avoid hardcoded path separators that break on Windows.
- **R-TOML-004** SHOULD: Centralize error handling for YAML/TOML parsing failures in utility functions to provide consistent error messages.
- **R-TOML-005** SHOULD: Document the expected structure of ParsedFrontmatter and CopilotToolMapping in TypeScript interfaces with JSDoc comments.

### Verify

```bash
# Verify @iarna/toml is used for TOML parsing in SubagentsProcessor.ts
grep -r "import.*from '@iarna/toml'" src/core/SubagentsProcessor.ts

# Verify fs/promises is imported in subagent processing modules
grep -r "import.*from 'fs/promises'" src/core/Subagents*.ts

# Verify js-yaml is imported in subagent processing modules
grep -r "import.*from 'js-yaml'" src/core/Subagents*.ts

# Verify shared utility functions are exported from SubagentsUtils.ts
grep -r "parseFrontmatter\|validateFrontmatter\|loadSubagentFile" src/core/SubagentsUtils.ts

# Verify no synchronous fs methods are used
grep -r "fs\.readFileSync\|fs\.writeFileSync" src/core/Subagents*.ts
```

**Accept when:**
- All subagent processing modules import fs/promises, js-yaml, and path from the standard library set
- @iarna/toml is imported and used in SubagentsProcessor.ts for TOML parsing
- Shared utility functions (parseFrontmatter, validateFrontmatter, loadSubagentFile) are exported from SubagentsUtils.ts and imported by consuming modules
- No synchronous fs methods (readFileSync, writeFileSync) are used in SubagentsUtils.ts or SubagentsProcessor.ts
- Type contracts (ParsedFrontmatter, CopilotToolMapping) are consistently used across modules

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep commands MUST be executed in CI pipeline to detect non-standard library imports. Code review MUST verify library imports in subagent-related PRs. TypeScript compilation MUST ensure shared type contracts are used consistently. CI pipeline MUST fail if non-standard parsing libraries are detected in subagent processing modules.
</enforcement>