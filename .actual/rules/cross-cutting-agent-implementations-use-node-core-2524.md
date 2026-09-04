# Use Node.js Core Modules for File System Operations in Agent Layer: Agent Implementations Use Node Core Modules

These rules are ALWAYS ACTIVE for all agent implementation files and VSCode integration code that perform file system operations, including path manipulation, file reading, directory access, and configuration file loading.

### Rules

- **R-AGENT-FS-001** MUST: Agent implementations MUST use Node.js core modules (path, fs, fs/promises) as the primary mechanism for file system operations including path manipulation, file reading, and directory access.
- **R-AGENT-FS-002** MUST: All path construction MUST use path.resolve() or path.join() rather than string concatenation to ensure cross-platform compatibility.
- **R-AGENT-FS-003** SHOULD: Prefer fs/promises with async/await over callback-based fs APIs for new code to avoid blocking the event loop.
- **R-AGENT-FS-004** MUST: File read operations and JSON parsing MUST be wrapped in try-catch blocks to handle both I/O errors and parse errors separately.
- **R-AGENT-FS-005** MUST: No third-party file system libraries (e.g., fs-extra, graceful-fs) MAY be imported in agent layer code without documented exception and architecture review approval.

### Verify

```bash
# Discover the project's static analysis configuration and execute import validation rules
# to confirm core module usage in agent implementations
grep -r "from ['\"]fs" src/agents/ src/vscode/ | grep -v "node:fs" | grep -v "fs/promises"

# Inspect agent implementation files to confirm imports match the required core module pattern
grep -r "import.*from.*['\"]fs" src/agents/ | head -20

# Verify no third-party file system libraries are imported in agent layer code
grep -r "from ['\"]fs-extra\|graceful-fs\|upath" src/agents/ src/vscode/

# Locate and run file system operation tests to verify core module integration
find . -name "*.test.ts" -o -name "*.spec.ts" | xargs grep -l "fs\|path" | head -10
```

**Accept when:**
- All agent implementation files import file system functionality exclusively from Node.js core modules (path, fs, fs/promises)
- Static analysis confirms no third-party file system libraries are imported in agent layer code
- All path construction uses path.resolve() or path.join() rather than string concatenation
- File read operations and JSON parsing are wrapped in try-catch blocks with separate error handling
- File system operation tests pass using the core module APIs

<enforcement>
Claude Code MUST NOT skip or defer verification. All file system operations in agent implementations MUST comply with these rules before code is committed.
</enforcement>