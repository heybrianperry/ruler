# Use Node.js Core Modules for File System Operations in VSCode Settings Management: Implementations Extend Cache

These rules are ALWAYS ACTIVE for all files in the VSCode settings management module (src/vscode/settings.ts) and related file system abstraction layers that handle MCP server configuration persistence.

### Rules

- **R-VSCODE-FS-001** MUST: Use Node.js core modules 'fs' and 'path' for all file system operations in VSCode settings management without introducing external database dependencies.
- **R-VSCODE-FS-002** MUST: Centralize all file system operations in the FileSystemUtils abstraction layer to enable consistent error handling and future storage backend changes.
- **R-VSCODE-FS-003** MUST: Wrap JSON.parse calls in try-catch blocks and validate parsed objects against expected schema before populating the cache.
- **R-VSCODE-FS-004** MUST: Use atomic write patterns (write to temp file, then rename) to prevent partial writes from corrupting configuration files.
- **R-VSCODE-FS-005** SHOULD: Implement file locking mechanism or atomic write-rename pattern to prevent concurrent writes from corrupting settings files.
- **R-VSCODE-FS-006** SHOULD: Implement file watching with fs.watch or document cache refresh requirements; consider TTL-based invalidation for cache staleness mitigation.
- **R-VSCODE-FS-007** MAY: Implementations MAY extend the cache layer with additional Map instances for different entity types beyond servers.
- **R-VSCODE-FS-008** MUST: Maintain in-memory Map-based caching (existingServerMap) to reduce redundant file I/O operations and improve read performance for frequently accessed configurations.
- **R-VSCODE-FS-009** MUST: Persist configuration data in JSON-formatted files that are human-readable and suitable for version control and manual editing.
- **R-VSCODE-FS-010** MUST: Export public API contracts (VSCodeSettings, AugmentMcpServer, readVSCodeSettings, writeVSCodeSettings) that abstract file system operations from consumers.

### Verify

```bash
# Verify fs and path core module imports are present
grep -r "require.*['\"]fs['\"]\|import.*from.*['\"]fs['\"]" src/vscode/settings.ts

# Verify JSON.parse usage for deserializing configuration
grep -r "JSON\.parse" src/vscode/settings.ts

# Verify Map data structure usage for cache layer
grep -r "existingServerMap\.set\|Map" src/vscode/settings.ts

# Verify public API functions are exported
grep -r "readVSCodeSettings\|writeVSCodeSettings" src/vscode/settings.ts

# Verify FileSystemUtils abstraction is used for file operations
grep -r "FileSystemUtils" src/vscode/settings.ts

# Verify atomic write patterns (temp file + rename)
grep -r "writeFileSync.*tmp\|renameSync" src/core/FileSystemUtils.ts

# Verify try-catch wrapping around JSON.parse
grep -B2 -A2 "JSON\.parse" src/vscode/settings.ts | grep -E "try|catch"
```

**Accept when:**
- Grep commands confirm presence of 'fs' and 'path' core module imports in src/vscode/settings.ts
- JSON.parse usage is detected for deserializing configuration content
- Map data structure usage with .set() method is confirmed for cache layer implementation
- Public API functions readVSCodeSettings and writeVSCodeSettings are exported from the module
- FileSystemUtils abstraction is used for all file system operations
- Atomic write patterns (write to temp file, then rename) are implemented in FileSystemUtils
- JSON.parse calls are wrapped in try-catch blocks with error handling
- No direct fs module usage exists outside the FileSystemUtils abstraction layer

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-VSCODE-FS rules MUST be verified before accepting changes to src/vscode/settings.ts or related file system abstraction layers. CI build MUST fail if verification commands do not detect required patterns. Code review MUST reject PRs introducing alternative storage mechanisms without architectural discussion and ADR amendment.
</enforcement>