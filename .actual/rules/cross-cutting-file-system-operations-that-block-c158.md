# Use Node.js Core Modules for File System Operations: File System Operations That Block Event

These rules are ALWAYS ACTIVE for all files matching the configured scope: agent implementations that read or write configuration files, CLI command handlers that access the file system, core utilities that perform path resolution or file system queries, state management components that persist or retrieve data from disk, and any component that needs to verify file existence, read file metadata, or manipulate file system paths.

### Rules

- **R-NODEJS-FS-001** SHOULD: File system operations that may block the event loop SHOULD use asynchronous APIs rather than synchronous variants except during application initialization or CLI command execution where blocking is acceptable.

### Verify

```bash
# Discover the project's static analysis configuration and execute the import analysis tool
# to verify that file system operations import from core modules
grep -r "require.*fs\|import.*from.*['\"]fs['\"]" --include="*.ts" --include="*.js" | head -20

# Search the codebase for synchronous file system operations outside initialization/CLI contexts
grep -r "readFileSync\|writeFileSync\|readdirSync" --include="*.ts" --include="*.js" | grep -v "// initialization\|// CLI" | head -20

# Verify path manipulation uses the path module
grep -r "path\.join\|path\.resolve" --include="*.ts" --include="*.js" | wc -l

# Check for string concatenation in file paths (potential violations)
grep -r "\+ ['\"].*\/" --include="*.ts" --include="*.js" | grep -v "http" | head -10
```

**Accept when:**
- Static analysis confirms all file system read, write, and metadata operations use Node.js core modules (fs, path)
- Path manipulation operations consistently use the path module across all components
- No direct string concatenation is used for path construction in file system operations
- Synchronous file system APIs are only used during application initialization or CLI command execution
- Promise-based async APIs are preferred over callback-based APIs for integration with async/await patterns

<enforcement>
Claude Code MUST NOT skip or defer verification. All file system operations must be reviewed for compliance with these rules before code is committed.
</enforcement>