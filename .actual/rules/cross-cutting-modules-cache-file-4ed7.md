# Adopt Async File System Operations as Standard I/O Pattern: Modules Cache File

These rules are ALWAYS ACTIVE for all file system operations across agents, configuration loaders, utility modules, MCP and VSCode settings management, and CLI handlers performing file operations.

### Rules

- **R-ASYNC-001** MAY: Modules MAY cache file system state (existingServerMap, seen Set) in memory when coordinating multiple related operations, provided cache invalidation is handled correctly.
- **R-ASYNC-002** MUST: All new file I/O code SHALL use async/await patterns or return Promises; synchronous file operations (readFileSync, writeFileSync) are prohibited except under documented exceptions.
- **R-ASYNC-003** MUST: All JSON.parse() calls SHALL be wrapped in try-catch blocks or error handling logic to prevent unhandled parse failures.
- **R-ASYNC-004** SHOULD: File system utilities SHALL export public contracts (findRulerDir, readMarkdownFiles, writeGeneratedFile, backupFile, walkSkillsTree, copySkillsDirectory) to centralize file operation logic and reduce code duplication.
- **R-ASYNC-005** SHOULD: Configuration updates SHALL implement the backupFile() pattern before modifying existing files; store backups with .backup extension.
- **R-ASYNC-006** SHOULD: Recursive operations (walkSkillsTree, findRulerDir) SHALL use async generators or batched processing to avoid loading entire directory trees into memory.
- **R-ASYNC-007** EXC-001: Synchronous file operations are permitted during module initialization or CLI startup where async context is not available; document with inline comment explaining necessity.
- **R-ASYNC-008** EXC-002: Synchronous file operations are permitted in performance-critical hot paths only after benchmarking demonstrates unacceptable latency with async operations; document with rationale.

### Verify

```bash
# Count synchronous file operations (should be minimal, <5 in src/ excluding tests)
grep -r "readFileSync\|writeFileSync" src/ --include="*.ts" | grep -v "test" | wc -l

# Count async file operation imports
grep -r "from 'fs/promises'\|from \"fs/promises\"" src/ --include="*.ts" | wc -l

# Verify JSON.parse() calls are wrapped in error handling
grep -r "JSON.parse" src/ --include="*.ts" | grep -v "try" -A 2 -B 2 | grep -c "catch"

# Count async file operation functions
grep -r "async.*function\|async .*=>" src/ --include="*.ts" | grep -E "(read|write|load|save|update|propagate|discover)" | wc -l
```

**Accept when:**
- Synchronous file operations (readFileSync, writeFileSync) are absent or minimal (<5 occurrences) in src/ directory excluding tests
- At least 80% of file operation functions use async/await or return Promises, verified by grep pattern matching
- All JSON.parse() calls are wrapped in try-catch blocks or error handling logic, verified by code review or static analysis
- Integration tests pass for concurrent agent configuration and skills propagation scenarios without race conditions or data corruption
- All new file I/O code uses 'fs/promises' import or wraps callback-based 'fs' functions in util.promisify()
- Backup-before-write pattern is implemented in all configuration update operations

<enforcement>
Clause Code MUST NOT skip or defer verification. All file system operations must be audited against these rules during code review. Pull requests introducing synchronous file operations outside documented exceptions must be flagged and justified. CI pipeline must fail if verification commands detect policy violations above threshold.
</enforcement>