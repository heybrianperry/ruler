# Adopt Async File System Operations as Standard I/O Pattern: Configuration Write Operations

These rules are ALWAYS ACTIVE for all file system operations across agents, configuration loaders, utility modules, MCP and VSCode settings management, and CLI handlers performing file operations.

### Rules

- **R-ASYNC-001** MUST: Configuration write operations MUST implement atomic write patterns (write to temp file, then rename) or backup existing files before modification.
- **R-ASYNC-002** MUST: Use 'fs/promises' import for new code; wrap callback-based 'fs' functions in util.promisify() for legacy compatibility.
- **R-ASYNC-003** MUST: Implement backupFile() pattern from FileSystemUtils before modifying existing configuration files; store backups with .backup extension.
- **R-ASYNC-004** MUST: Use try-catch blocks around JSON.parse() and file operations; provide meaningful error messages including file paths and operation context.
- **R-ASYNC-005** SHOULD: For recursive operations (walkSkillsTree, findRulerDir), use async generators or batched processing to avoid loading entire directory trees into memory.
- **R-ASYNC-006** SHOULD: Export file system functions as public contracts in utility modules; avoid inline file operations in agent or handler code.
- **R-ASYNC-007** MAY: Synchronous file operations are permitted during module initialization or CLI startup where async context is not available (EXC-001).
- **R-ASYNC-008** MAY: Synchronous file operations are permitted in performance-critical hot paths after benchmarking demonstrates unacceptable latency with async operations (EXC-002).

### Verify

```bash
# Count synchronous file operations (should be minimal, <5 in src/)
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
Claude Code MUST NOT skip or defer verification. All file system write operations MUST be reviewed against R-ASYNC-001 through R-ASYNC-006 before approval. Exceptions require inline documentation and tech lead review per the exception process defined in the ADR.
</enforcement>