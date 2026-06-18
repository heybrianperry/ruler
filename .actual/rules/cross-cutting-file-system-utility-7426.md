# Adopt Async File System Operations as Standard I/O Pattern: File System Utility

These rules are ALWAYS ACTIVE for all file system operations across agents, configuration loaders, utility modules, MCP and VSCode settings management, and CLI handlers performing file operations.

### Rules

- **R-ASYNC-FS-001** SHOULD: File system utility functions SHOULD be centralized in dedicated modules (FileSystemUtils, SkillsUtils) and exported as public contracts for reuse across agents.
- **R-ASYNC-FS-002** MUST: Use 'fs/promises' import for new code; wrap callback-based 'fs' functions in util.promisify() for legacy compatibility.
- **R-ASYNC-FS-003** MUST: Implement backupFile() pattern from FileSystemUtils before modifying existing configuration files; store backups with .backup extension.
- **R-ASYNC-FS-004** MUST: Use try-catch blocks around JSON.parse() and file operations; provide meaningful error messages including file paths and operation context.
- **R-ASYNC-FS-005** SHOULD: For recursive operations (walkSkillsTree, findRulerDir), use async generators or batched processing to avoid loading entire directory trees into memory.
- **R-ASYNC-FS-006** MUST: Export file system functions as public contracts in utility modules; avoid inline file operations in agent or handler code.
- **R-ASYNC-FS-EXC-001** MAY: Synchronous file operations are permitted during module initialization or CLI startup where async context is not available, with inline comment explaining the constraint.
- **R-ASYNC-FS-EXC-002** MAY: Performance-critical hot paths may use synchronous reads after benchmarking demonstrates unacceptable latency with async operations, with documentation of performance data.

### Verify

```bash
# Count synchronous file operations (should be minimal, <5 in src/ excluding tests)
grep -r "readFileSync\|writeFileSync" src/ --include="*.ts" | grep -v "test" | wc -l

# Count async file operation imports
grep -r "from 'fs/promises'\|from \"fs/promises\"" src/ --include="*.ts" | wc -l

# Verify JSON.parse() calls are wrapped in try-catch
grep -r "JSON.parse" src/ --include="*.ts" | grep -v "try" -A 2 -B 2 | grep -c "catch"

# Count async file operation functions
grep -r "async.*function\|async .*=>" src/ --include="*.ts" | grep -E "(read|write|load|save|update|propagate|discover)" | wc -l
```

**Accept when:**
- Synchronous file operations (readFileSync, writeFileSync) are absent or minimal (<5 occurrences) in src/ directory excluding tests
- At least 80% of file operation functions use async/await or return Promises, verified by grep pattern matching
- All JSON.parse() calls are wrapped in try-catch blocks or error handling logic, verified by code review or static analysis
- Integration tests pass for concurrent agent configuration and skills propagation scenarios without race conditions or data corruption

<enforcement>
Clause Code MUST NOT skip or defer verification. All new file I/O code must be reviewed against the async/await requirement. Static analysis rules MUST detect synchronous fs module usage outside approved exceptions. CI pipeline MUST fail if verification commands detect policy violations above threshold.
</enforcement>