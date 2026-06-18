# Adopt Async File System Operations as Standard I/O Pattern: Path Resolution Use

These rules are ALWAYS ACTIVE for all file system operations across agents, configuration loaders, utility modules, MCP and VSCode settings management, and CLI handlers performing file operations.

### Rules

- **R-ASYNC-PATH-001** MUST: Path resolution MUST use 'path' module functions (join, resolve, dirname) to ensure cross-platform compatibility.

### Verify

```bash
# Check for synchronous file operations outside approved exceptions
grep -r "readFileSync\|writeFileSync" src/ --include="*.ts" | grep -v "test" | wc -l

# Verify fs/promises imports are in use
grep -r "from 'fs/promises'\|from \"fs/promises\"" src/ --include="*.ts" | wc -l

# Verify JSON.parse calls have error handling
grep -r "JSON.parse" src/ --include="*.ts" | grep -v "try" -A 2 -B 2 | grep -c "catch"

# Count async file operation functions
grep -r "async.*function\|async .*=>" src/ --include="*.ts" | grep -E "(read|write|load|save|update|propagate|discover)" | wc -l

# Verify path module usage in file operations
grep -r "path\.join\|path\.resolve\|path\.dirname" src/ --include="*.ts" | grep -E "(read|write|load|save|update|propagate|discover)" | wc -l
```

**Accept when:**
- Synchronous file operations (readFileSync, writeFileSync) are absent or minimal (<5 occurrences) in src/ directory excluding tests
- At least 80% of file operation functions use async/await or return Promises
- All JSON.parse() calls are wrapped in try-catch blocks or error handling logic
- Integration tests pass for concurrent agent configuration and skills propagation scenarios without race conditions or data corruption
- Path module functions (join, resolve, dirname) are used consistently in all file path construction within file operation contexts

<enforcement>
Claude Code MUST NOT skip or defer verification. All file system operations must comply with path resolution requirements before merge.
</enforcement>