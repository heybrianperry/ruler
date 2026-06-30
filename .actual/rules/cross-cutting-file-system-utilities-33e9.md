# Use Node.js Core Modules for File System Operations in VSCode Settings Management: File System Utilities

These rules are ALWAYS ACTIVE for all file system operations in VSCode settings management modules, particularly src/vscode/settings.ts and ../core/FileSystemUtils, ensuring consistent use of Node.js core modules for configuration persistence without external database dependencies.

### Rules

- **R-FSUTIL-001** MUST: Use Node.js core modules 'fs' and 'path' for all file system operations in VSCode settings management.
- **R-FSUTIL-002** SHOULD: Centralize all file system utilities in ../core/FileSystemUtils module for reuse across components.
- **R-FSUTIL-003** MUST: Wrap JSON.parse calls in try-catch blocks and validate parsed objects against expected schema before populating cache.
- **R-FSUTIL-004** SHOULD: Use atomic write patterns (write to temp file, then rename) to prevent partial writes from corrupting configuration files.
- **R-FSUTIL-005** MUST: Maintain in-memory Map-based caching of server instances by name to reduce redundant file I/O operations.
- **R-FSUTIL-006** SHOULD: Implement file locking mechanism or atomic write-rename pattern in FileSystemUtils to prevent concurrent write corruption.
- **R-FSUTIL-007** SHOULD: Document cache invalidation strategy and provide explicit refresh methods if external modification of settings files is supported.
- **R-FSUTIL-008** MUST: Store configuration data in JSON-formatted files only; do not use binary or non-JSON serialization formats.

### Verify

```bash
# Verify fs and path core module imports are present
grep -r "require.*['\"]fs['\"]"|import.*from.*['\"]fs['\"]" src/vscode/settings.ts

# Verify JSON.parse usage for deserializing configuration
grep -r "JSON\.parse" src/vscode/settings.ts

# Verify Map data structure usage for cache layer
grep -r "existingServerMap\.set\|Map" src/vscode/settings.ts

# Verify public API functions are exported
grep -r "readVSCodeSettings\|writeVSCodeSettings" src/vscode/settings.ts

# Verify FileSystemUtils abstraction exists and is used
grep -r "FileSystemUtils" src/vscode/settings.ts
```

**Accept when:**
- Grep commands confirm presence of 'fs' and 'path' core module imports in src/vscode/settings.ts
- JSON.parse usage is detected for deserializing configuration content
- Map data structure usage with .set() method is confirmed for cache layer implementation
- Public API functions readVSCodeSettings and writeVSCodeSettings are exported from the module
- FileSystemUtils abstraction is used for all file operations, with no direct fs module calls outside the abstraction layer
- All JSON.parse calls are wrapped in try-catch blocks with error handling
- Atomic write patterns are implemented in FileSystemUtils for configuration persistence

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-FSUTIL rules MUST be verified before accepting changes to file system operations in VSCode settings management modules. Violations detected by grep commands or code review MUST result in rejection of the change.
</enforcement>