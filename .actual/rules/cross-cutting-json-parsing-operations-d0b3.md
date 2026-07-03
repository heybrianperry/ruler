# Adopt Async Function Orchestration for Agent Configuration and Skill Management: Json Parsing Operations

These rules are ALWAYS ACTIVE for all agent configuration and skill processing operations, including FirebenderAgent configuration management, SkillsProcessor workflows, and any coordinated file system operations using Node.js fs/fs.promises APIs.

### Rules

- **R-ASYNC-001** SHOULD: JSON parsing operations SHOULD validate input and handle parse errors gracefully with logged warnings.

### Verify

```bash
# Verify fs.promises usage across agent and core modules
grep -r "fs\.promises\|from 'fs/promises'" src/agents src/core --include='*.ts' | wc -l

# Verify named async functions in key modules
grep -r "async function\|async .*=>" src/agents/FirebenderAgent.ts src/core/SkillsProcessor.ts | wc -l

# Verify JSON.parse error handling coverage
grep -r "JSON\.parse" src/agents src/core --include='*.ts' -A 2 | grep -c "try\|catch"
```

**Accept when:**
- All file I/O operations in FirebenderAgent and SkillsProcessor use fs.promises or equivalent promise-based APIs
- Complex async workflows are decomposed into at least 3 named async functions per module
- At least 80% of JSON.parse operations are wrapped in try-catch blocks with error handling

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST enforce async/await pattern verification. ESLint rules MUST enforce promise-based fs APIs and async function naming conventions. Integration tests MUST cover async error paths and workflow coordination. CI pipeline MUST fail if callback-based fs APIs are detected in new code.
</enforcement>