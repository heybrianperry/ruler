# Adopt Async File System Operations as Standard I/O Pattern: Configuration Read Operations

These rules are ALWAYS ACTIVE for all file system operations across agents, configuration loaders, utility modules, MCP and VSCode settings management, and CLI handlers performing file operations.

### Rules

- **R-ASYNC-001** MUST: Configuration read operations MUST parse JSON using JSON.parse() with try-catch error handling and provide fallback behavior for missing or malformed files.

### Verify

```bash
# Check for synchronous file operations outside approved exceptions
grep -r "readFileSync\|writeFileSync" src/ --include="*.ts" | grep -v "test" | wc -l

# Verify fs/promises imports are in use
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
Clause Code MUST NOT skip or defer verification. All new file I/O code must use async operations with proper error handling. Violations are flagged for review and justification.
</enforcement>