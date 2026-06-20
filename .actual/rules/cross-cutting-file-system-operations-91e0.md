# Adopt Async Function Orchestration for Agent Configuration and Skill Management: File System Operations

These rules are ALWAYS ACTIVE for all agent configuration and skill processing operations, including FirebenderAgent configuration loading, parsing, and persistence; SkillsProcessor discovery, cleanup, and propagation workflows; and any agent or core module performing coordinated file system operations.

### Rules

- **R-FSO-001** MUST: All file system I/O operations in agent configuration and skill processing MUST use async/await with fs.promises or equivalent promise-based APIs.
- **R-FSO-002** MUST: Name async functions descriptively based on their workflow purpose (e.g., loadExistingConfig, propagateSkillsForClaude) to improve code navigation and testing.
- **R-FSO-003** MUST: Wrap JSON.parse calls in try-catch blocks and provide sensible defaults or log warnings for parse failures.
- **R-FSO-004** SHOULD: Use Set or Map for deduplication in async workflows to prevent duplicate processing; ensure state mutations complete before dependent operations.
- **R-FSO-005** SHOULD: Consider Promise.all for independent async operations (e.g., propagating to multiple targets) to improve performance.
- **R-FSO-006** MUST: Implement comprehensive try-catch blocks in all async functions to prevent unhandled promise rejections.

### Verify

```bash
# Count fs.promises usage across agent and core modules
grep -r "fs\.promises\|from 'fs/promises'" src/agents src/core --include='*.ts' | wc -l

# Count named async functions in FirebenderAgent and SkillsProcessor
grep -r "async function\|async .*=>" src/agents/FirebenderAgent.ts src/core/SkillsProcessor.ts | wc -l

# Count JSON.parse operations with try-catch error handling
grep -r "JSON\.parse" src/agents src/core --include='*.ts' -A 2 | grep -c "try\|catch"
```

**Accept when:**
- All file I/O operations in FirebenderAgent and SkillsProcessor use fs.promises or equivalent promise-based APIs
- Complex async workflows are decomposed into at least 3 named async functions per module
- At least 80% of JSON.parse operations are wrapped in try-catch blocks with error handling
- No callback-based fs APIs (fs.readFile, fs.writeFile without promises) are detected in new code
- All async functions include proper error handling and do not produce unhandled promise rejections

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if async functions lack proper error handling. CI pipeline MUST fail if callback-based fs APIs are detected in new code.
</enforcement>