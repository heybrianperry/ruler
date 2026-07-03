# Adopt Async Function Orchestration for Agent Configuration and Skill Management: Error Handling Async

These rules are ALWAYS ACTIVE for all agent configuration and skill processing operations, including FirebenderAgent configuration management, SkillsProcessor discovery and propagation workflows, and any agent or core module performing coordinated file system operations.

### Rules

- **R-ASYNC-001** SHOULD: Error handling for async file operations SHOULD use try-catch blocks with fallback behavior (e.g., console.warn for non-critical failures, default values for missing configs).
- **R-ASYNC-002** MUST: Use fs.promises for all file system operations; import as `import { promises as fs } from "fs"` or `import fs from "fs/promises"`.
- **R-ASYNC-003** SHOULD: Name async functions descriptively based on their workflow purpose (e.g., loadExistingConfig, propagateSkillsForClaude) to improve code navigation and testing.
- **R-ASYNC-004** MUST: Wrap JSON.parse calls in try-catch blocks and provide sensible defaults or log warnings for parse failures.
- **R-ASYNC-005** SHOULD: Use Set or Map for deduplication in async workflows to prevent duplicate processing; ensure state mutations complete before dependent operations.
- **R-ASYNC-006** MAY: Consider Promise.all for independent async operations (e.g., propagating to multiple targets) to improve performance.

### Verify

```bash
# Count fs.promises usage across agent and core modules
grep -r "fs\.promises\|from 'fs/promises'" src/agents src/core --include='*.ts' | wc -l

# Count async function declarations in key modules
grep -r "async function\|async .*=>" src/agents/FirebenderAgent.ts src/core/SkillsProcessor.ts | wc -l

# Count JSON.parse operations with try-catch error handling
grep -r "JSON\.parse" src/agents src/core --include='*.ts' -A 2 | grep -c "try\|catch"
```

**Accept when:**
- All file I/O operations in FirebenderAgent and SkillsProcessor use fs.promises or equivalent promise-based APIs
- Complex async workflows are decomposed into at least 3 named async functions per module
- At least 80% of JSON.parse operations are wrapped in try-catch blocks with error handling

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST check async/await pattern compliance. ESLint rules MUST enforce promise-based fs APIs and async function naming. CI pipeline MUST fail if callback-based fs APIs are detected in new code.
</enforcement>