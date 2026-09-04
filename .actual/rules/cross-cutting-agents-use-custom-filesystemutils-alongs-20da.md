# Use Node.js Core Modules for File System Operations in Agent Layer: Agents Use Custom Filesystemutils Alongside Core

These rules are ALWAYS ACTIVE for all agent implementation files and VSCode integration code requiring file system access, including code reading configuration files, MCP definitions, settings from the local file system, and modules performing path resolution or directory traversal.

### Rules

- **R-FSCORE-001** MUST: Use Node.js core modules (path, fs, fs/promises) exclusively for file system operations in agent layer code.
- **R-FSCORE-002** MUST: Prefer fs/promises with async/await over callback-based fs APIs to avoid blocking the event loop during agent initialization and I/O operations.
- **R-FSCORE-003** MUST: Use path.resolve() or path.join() with appropriate base paths for all file path construction rather than string concatenation to ensure cross-platform compatibility.
- **R-FSCORE-004** MUST: Wrap file read operations and JSON parsing in separate try-catch blocks to handle both I/O errors and parse errors distinctly.
- **R-FSCORE-005** MAY: Agents MAY use custom FileSystemUtils alongside core modules to provide project-specific abstractions over file system primitives.
- **R-FSCORE-006** SHOULD: Establish and document conventions for when synchronous fs APIs are acceptable (e.g., CLI startup code or build scripts where blocking is acceptable).
- **R-FSCORE-007** SHOULD: Verify error handling for all file I/O operations to prevent unhandled ENOENT, EACCES, or permission errors.

### Verify

```bash
# Discover the project's static analysis configuration and execute import validation rules
# to confirm core module usage in agent layer
grep -r "from ['\"]fs['\"]\|from ['\"]path['\"]\|from ['\"]fs/promises['\"]" src/agents/ || echo "Checking for core module imports..."

# Locate and run the project's test suite for file system operations
npm test -- --testPathPattern="(agent|filesystem)" 2>/dev/null || echo "Run file system operation tests"

# Inspect agent implementation files to confirm imports match core module pattern
find src/agents -name "*.ts" -o -name "*.js" | xargs grep -l "require.*fs\|import.*fs" | head -20

# Automated dependency analysis to flag third-party file system libraries
grep -E "(glob|globby|fast-glob|chokidar|fs-extra|graceful-fs)" package.json package-lock.json yarn.lock 2>/dev/null || echo "No third-party fs libraries detected"
```

**Accept when:**
- All agent implementation files import file system functionality exclusively from Node.js core modules (path, fs, fs/promises)
- Static analysis confirms no third-party file system libraries are imported in agent layer code
- File system operation tests pass using the core module APIs
- All file path operations use path.resolve() or path.join() rather than string concatenation
- File read and JSON parse operations are wrapped in separate try-catch blocks

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tooling MUST scan import statements in agent layer modules. Code review MUST verify core module usage for new file I/O operations. CI pipeline MUST fail if third-party file system libraries are detected in agent layer dependencies.
</enforcement>