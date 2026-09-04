# Use Node.js Core Modules for File System Operations in Agent Layer: Agent Code Prefer Promises Based Asynchronous

These rules are ALWAYS ACTIVE for all agent implementation classes in the agent layer, VSCode integration code requiring file system access, code reading configuration files or MCP definitions from the local file system, and modules performing path resolution or directory traversal.

### Rules

- **R-AGENT-FS-001** SHOULD: Agent code SHOULD prefer the promises-based API (fs/promises) for asynchronous file operations to maintain consistency with async/await patterns.
- **R-AGENT-FS-002** MUST: Use path.resolve() or path.join() with appropriate base paths rather than string concatenation to construct file paths, ensuring cross-platform compatibility.
- **R-AGENT-FS-003** MUST: Wrap file read operations and JSON parsing in try-catch blocks to handle both I/O errors and parse errors separately when reading JSON configuration files.
- **R-AGENT-FS-004** MUST: Agent implementations MUST import file system functionality exclusively from Node.js core modules (path, fs, fs/promises) and not from third-party file system libraries.
- **R-AGENT-FS-005** SHOULD: For agent initialization code that reads configuration, prefer fs/promises with async/await to avoid blocking the event loop during startup.

### Verify

```bash
# Discover the project's static analysis configuration and execute the import validation rules
# to confirm core module usage
grep -r "from ['\"]fs" --include="*.ts" --include="*.js" src/agent/ | grep -v "fs/promises" | grep -v "node:fs"

# Locate the project's test suite and run file system operation tests
npm test -- --testPathPattern=".*\\.test\\.(ts|js)$" --testNamePattern="file.*system|fs"

# Inspect agent implementation files to confirm imports match the required core module pattern
grep -r "import.*from.*['\"]fs" src/agent/ | grep -E "(path|fs|fs/promises)"

# Verify no third-party file system libraries are imported in agent layer
grep -r "import.*from.*['\"]" src/agent/ | grep -v "node:" | grep -v "fs" | grep -v "path" | grep -v "@" | grep -v "\./" | grep -v "../"
```

**Accept when:**
- All agent implementation files import file system functionality exclusively from Node.js core modules (path, fs, fs/promises)
- Static analysis confirms no third-party file system libraries are imported in agent layer code
- File system operation tests pass using the core module APIs
- All file path construction uses path.resolve() or path.join() rather than string concatenation
- JSON configuration file reads are wrapped in try-catch blocks with separate error handling

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for agent layer code. Violations must be caught during code review and static analysis before merge.
</enforcement>