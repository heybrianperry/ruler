# Use Node.js Core Modules for File System Operations in Agent Layer: Path Manipulation Operations Use Module Rather

These rules are ALWAYS ACTIVE for all agent implementation classes in the agent layer, VSCode integration code requiring file system access, code reading configuration files or MCP definitions from the local file system, and modules performing path resolution or directory traversal.

### Rules

- **R-FSOP-001** SHOULD: Path manipulation operations SHOULD use the path module rather than string concatenation to ensure cross-platform compatibility.
- **R-FSOP-002** MUST: All agent implementation files import file system functionality exclusively from Node.js core modules (path, fs, fs/promises).
- **R-FSOP-003** SHOULD: Prefer fs/promises with async/await for agent initialization code that reads configuration to avoid blocking the event loop during startup.
- **R-FSOP-004** MUST: Wrap file read operations and JSON parsing in try-catch blocks to handle both I/O errors and parse errors separately when reading JSON configuration files.
- **R-FSOP-005** SHOULD: Use path.resolve() or path.join() with appropriate base paths rather than string concatenation to construct file paths, ensuring cross-platform compatibility.

### Verify

```bash
# Discover the project's static analysis configuration and execute the import validation rules
# to confirm core module usage
grep -r "from ['\"]path['\"]" src/agent/ || echo "Path module imports found"
grep -r "from ['\"]fs['\"]" src/agent/ || echo "FS module imports found"
grep -r "from ['\"]fs/promises['\"]" src/agent/ || echo "FS/promises module imports found"

# Verify no third-party file system libraries are imported in agent layer code
grep -r "from ['\"][^'\"]*fs[^'\"]*['\"]" src/agent/ | grep -v "^.*from ['\"]fs" | grep -v "^.*from ['\"]path" && echo "WARNING: Third-party FS libraries detected" || echo "No third-party FS libraries found"

# Run file system operation tests to verify core module integration
npm test -- --testPathPattern="(agent|filesystem)" 2>/dev/null || echo "Test suite execution required"

# Inspect agent implementation files to confirm imports match the required core module pattern
find src/agent -name "*.ts" -o -name "*.js" | xargs grep -l "require.*fs\|import.*fs" | head -20
```

**Accept when:**
- All agent implementation files import file system functionality exclusively from Node.js core modules (path, fs, fs/promises)
- Static analysis confirms no third-party file system libraries are imported in agent layer code
- File system operation tests pass using the core module APIs
- Path manipulation operations use path.resolve() or path.join() instead of string concatenation
- JSON configuration file reads are wrapped in try-catch blocks handling both I/O and parse errors

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tooling scanning import statements in agent layer modules is mandatory. Code review checklist verifying core module usage for new file I/O operations is mandatory. Automated dependency analysis flagging third-party file system libraries in agent dependencies is mandatory.
</enforcement>