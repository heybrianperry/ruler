# Use Node.js Core Modules for File System Operations in VSCode Settings Management: File System Operations

These rules are ALWAYS ACTIVE for all file system operations within the VSCode settings management module (src/vscode/settings.ts) and related configuration persistence code.

### Rules

- **R-VSCODE-FS-001** MUST: File system operations for VSCode settings persistence MUST use Node.js core modules 'fs' and 'path' as detected in src/vscode/settings.ts
- **R-VSCODE-FS-002** MUST: All file system operations MUST be centralized in ../core/FileSystemUtils to enable consistent error handling and future storage backend changes
- **R-VSCODE-FS-003** MUST: JSON.parse calls MUST be wrapped in try-catch blocks and validated against expected schema before populating cache
- **R-VSCODE-FS-004** MUST: Atomic write patterns (write to temp file, then rename) MUST be used to prevent partial writes from corrupting configuration files
- **R-VSCODE-FS-005** SHOULD: In-memory Map-based caching of server instances by name SHOULD be maintained to reduce redundant file I/O operations
- **R-VSCODE-FS-006** SHOULD: Cache invalidation strategy SHOULD be documented and explicit refresh methods SHOULD be provided if external modification of settings files is supported

### Verify

```bash
# Verify fs and path core module imports in settings.ts
grep -r "require.*['\"]fs['\"]"|import.*from.*['\"]fs['\"]\"|require.*['\"]path['\"]"|import.*from.*['\"]path['\"]\"]" src/vscode/settings.ts

# Verify JSON.parse usage for deserializing configuration
grep -r "JSON\\.parse" src/vscode/settings.ts

# Verify Map data structure usage for cache layer
grep -r "existingServerMap\\.set\|new Map" src/vscode/settings.ts

# Verify public API functions are exported
grep -r "readVSCodeSettings\|writeVSCodeSettings" src/vscode/settings.ts

# Verify FileSystemUtils abstraction is used
grep -r "FileSystemUtils" src/vscode/settings.ts
```

**Accept when:**
- Grep commands confirm presence of 'fs' and 'path' core module imports in src/vscode/settings.ts
- JSON.parse usage is detected for deserializing configuration content
- Map data structure usage with .set() method is confirmed for cache layer implementation
- Public API functions readVSCodeSettings and writeVSCodeSettings are exported from the module
- FileSystemUtils abstraction is used for all file operations
- Try-catch blocks wrap JSON.parse calls
- Atomic write patterns are implemented in FileSystemUtils

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-VSCODE-FS rules are mandatory for VSCode settings file system operations. Violations must be caught during code review and CI pipeline checks.
</enforcement>