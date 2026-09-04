# Use Node.js Core Modules for File System Operations in Agent Layer: Code Performing File Import Core Modules

These rules are ALWAYS ACTIVE for all agent implementation classes, VSCode integration code, and any code reading configuration files, MCP definitions, or settings from the local file system.

### Rules

- **R-CORE-001** MUST: Code performing file I/O MUST import core modules using string literals matching the Node.js module resolution convention (e.g., `require('fs')`, `require('path')`, `require('fs/promises')`).
- **R-CORE-002** MUST: All path operations MUST use `path.resolve()` or `path.join()` rather than string concatenation to ensure cross-platform compatibility.
- **R-CORE-003** MUST: File read operations and JSON parsing MUST be wrapped in try-catch blocks to handle both I/O errors and parse errors separately.
- **R-CORE-004** SHOULD: New code performing file I/O SHOULD prefer `fs/promises` with async/await over callback-based `fs` APIs to avoid blocking the event loop.
- **R-CORE-005** MUST: Agent initialization code that reads configuration MUST use `fs/promises` with async/await to prevent blocking during startup.
- **R-CORE-006** MUST: No third-party file system libraries (e.g., `fs-extra`, `graceful-fs`) MUST be imported in agent layer code without documented exception and architecture review approval.

### Verify

```bash
# Discover the project's static analysis configuration and execute import validation rules
# to confirm core module usage in agent layer files
grep -r "require\|import" src/agent/ | grep -E "(fs|path)" | grep -v "node_modules"

# Locate and run the project's test suite for file system operations
npm test -- --testPathPattern="(file|fs|agent)" 2>/dev/null || echo "(no verify commands defined)"

# Inspect agent implementation files to confirm imports match the required core module pattern
find src/agent -name "*.ts" -o -name "*.js" | xargs grep -l "require\|import" | head -5

# Verify no third-party file system libraries are present in dependencies
grep -E "(fs-extra|graceful-fs|upath)" package.json package-lock.json 2>/dev/null || echo "No third-party fs libraries detected"
```

**Accept when:**
- All agent implementation files import file system functionality exclusively from Node.js core modules (`path`, `fs`, `fs/promises`)
- Static analysis confirms no third-party file system libraries are imported in agent layer code
- File system operation tests pass using the core module APIs
- All path operations use `path.resolve()` or `path.join()` for construction
- File read and JSON parse operations are wrapped in try-catch blocks
- Agent initialization code uses `fs/promises` with async/await

<enforcement>
Claude Code MUST NOT skip or defer verification. All file I/O operations in the agent layer MUST comply with these rules before code is committed.
</enforcement>