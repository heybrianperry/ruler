# Adopt Async Function Orchestration for Agent Configuration and Skill Management: Independent Async Operations

These rules are ALWAYS ACTIVE for all agent configuration and skill processing operations, including FirebenderAgent configuration loading, parsing, and persistence; SkillsProcessor discovery, cleanup, and propagation workflows; any agent or core module performing coordinated file system operations; and MCP (Model Context Protocol) configuration handling.

### Rules

- **R-ASYNC-001** MUST: Use fs.promises or equivalent promise-based APIs for all file system operations; import as `import { promises as fs } from "fs"` or `import fs from "fs/promises"`.
- **R-ASYNC-002** MUST: Decompose complex async workflows into named async functions (e.g., loadExistingConfig, propagateSkillsForClaude) rather than inline promises or callbacks.
- **R-ASYNC-003** MUST: Wrap all JSON.parse calls in try-catch blocks and provide sensible defaults or log warnings for parse failures.
- **R-ASYNC-004** SHOULD: Use Set or Map for deduplication in async workflows to prevent duplicate processing; ensure state mutations complete before dependent operations.
- **R-ASYNC-005** MAY: Independent async operations (e.g., propagating to multiple targets) MAY be executed in parallel using Promise.all or similar constructs.
- **R-ASYNC-006** MUST: Implement comprehensive try-catch blocks in all async functions to prevent unhandled promise rejections.

### Verify

```bash
# Count promise-based fs API usage
grep -r "fs\.promises\|from 'fs/promises'" src/agents src/core --include='*.ts' | wc -l

# Count named async functions in key modules
grep -r "async function\|async .*=>" src/agents/FirebenderAgent.ts src/core/SkillsProcessor.ts | wc -l

# Count JSON.parse operations with error handling
grep -r "JSON\.parse" src/agents src/core --include='*.ts' -A 2 | grep -c "try\|catch"
```

**Accept when:**
- All file I/O operations in FirebenderAgent and SkillsProcessor use fs.promises or equivalent promise-based APIs
- Complex async workflows are decomposed into at least 3 named async functions per module
- At least 80% of JSON.parse operations are wrapped in try-catch blocks with error handling
- All async functions include comprehensive error handling with try-catch blocks

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST check async/await pattern compliance. ESLint rules MUST enforce promise-based fs APIs and async function naming. CI pipeline MUST fail if callback-based fs APIs are detected in new code. Code review MUST block merge if async functions lack proper error handling.
</enforcement>