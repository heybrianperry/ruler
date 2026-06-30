# Standardize Core Library Imports for Subagent File Processing: Yaml Parsing Performed

These rules are ALWAYS ACTIVE for all modules in src/core/ that process subagent files, specifically SubagentsUtils.ts and SubagentsProcessor.ts, and all functions that parse, validate, discover, or write subagent configuration files.

### Rules

- **R-YAML-001** MUST: YAML parsing MUST be performed using the 'js-yaml' library for consistency across all subagent configuration files.
- **R-YAML-002** MUST: Import 'fs/promises' as a named import (e.g., `import { readFile, writeFile } from 'fs/promises'`) for tree-shaking benefits.
- **R-YAML-003** MUST: Use path.join() and path.resolve() consistently to avoid hardcoded path separators that break on Windows.
- **R-YAML-004** MUST: No synchronous fs methods are permitted in SubagentsUtils.ts or SubagentsProcessor.ts.
- **R-YAML-005** SHOULD: Centralize error handling for YAML/TOML parsing failures in utility functions to provide consistent error messages.
- **R-YAML-006** SHOULD: Document the expected structure of ParsedFrontmatter and CopilotToolMapping in TypeScript interfaces with JSDoc comments.

### Verify

```bash
# Verify js-yaml is used for YAML parsing
grep -r "import.*from 'js-yaml'" src/core/Subagents*.ts

# Verify fs/promises is imported
grep -r "import.*from 'fs/promises'" src/core/Subagents*.ts

# Verify @iarna/toml is used where needed
grep -r "import.*from '@iarna/toml'" src/core/SubagentsProcessor.ts

# Verify shared utility functions are exported and used
grep -r "parseFrontmatter\|validateFrontmatter\|loadSubagentFile" src/core/SubagentsUtils.ts

# Verify no synchronous fs methods are used
! grep -r "fs\.readFileSync\|fs\.writeFileSync\|fs\.readdirSync" src/core/Subagents*.ts
```

**Accept when:**
- All subagent processing modules import fs/promises, js-yaml, and path from the standard library set
- Shared utility functions (parseFrontmatter, validateFrontmatter, loadSubagentFile) are exported from SubagentsUtils.ts and imported by consuming modules
- No synchronous fs methods are used in SubagentsUtils.ts or SubagentsProcessor.ts
- All YAML parsing uses the js-yaml library exclusively
- Path operations use path.join() or path.resolve() consistently

<enforcement>
Claude Code MUST NOT skip or defer verification. All verify commands MUST pass before accepting changes to subagent file processing modules.
</enforcement>