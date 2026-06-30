# Use Node.js Core Modules for File System Operations in VSCode Settings Management: Input Validation Performed

These rules are ALWAYS ACTIVE for all file system operations and configuration management code in the VSCode settings module (src/vscode/settings.ts) and related FileSystemUtils abstractions.

### Rules

- **R-VSCODE-FS-001** MUST: Use Node.js core modules 'fs' and 'path' for all file system operations in VSCode settings management without introducing external database dependencies.
- **R-VSCODE-FS-002** SHOULD: Input validation SHOULD be performed on parsed JSON content to prevent injection or malformed data issues.
- **R-VSCODE-FS-003** MUST: Wrap all JSON.parse calls in try-catch blocks and validate parsed objects against expected schema before populating cache.
- **R-VSCODE-FS-004** MUST: Centralize all file system operations in ../core/FileSystemUtils to enable consistent error handling and future storage backend changes.
- **R-VSCODE-FS-005** SHOULD: Use atomic write patterns (write to temp file, then rename) to prevent partial writes from corrupting configuration files.
- **R-VSCODE-FS-006** MUST: Maintain in-memory Map-based caching (existingServerMap) to track server instances and avoid redundant file operations.
- **R-VSCODE-FS-007** SHOULD: Implement file locking mechanism or atomic write-rename pattern to mitigate concurrent write corruption risks.
- **R-VSCODE-FS-008** SHOULD: Document cache invalidation strategy and provide explicit refresh methods if external modification of settings files is supported.

### Verify

```bash
# Verify fs and path core module imports are present
grep -r "require.*['\"]fs['\"]"|import.*from.*['\"]fs['\"]\"|require.*['\"]path['\"]"|import.*from.*['\"]path['\"]\"
 src/vscode/settings.ts

# Verify JSON.parse usage for deserializing configuration
grep -r "JSON\.parse" src/vscode/settings.ts

# Verify Map data structure usage for cache layer
grep -r "existingServerMap\.set\|new Map" src/vscode/settings.ts

# Verify public API functions are exported
grep -r "readVSCodeSettings\|writeVSCodeSettings" src/vscode/settings.ts

# Verify FileSystemUtils abstraction is used
grep -r "FileSystemUtils" src/vscode/settings.ts

# Verify try-catch wrapping around JSON.parse
grep -r "try\|catch" src/vscode/settings.ts | grep -A 2 "JSON\.parse"
```

**Accept when:**
- Grep commands confirm presence of 'fs' and 'path' core module imports in src/vscode/settings.ts
- JSON.parse usage is detected for deserializing configuration content
- Map data structure usage with .set() method is confirmed for cache layer implementation
- Public API functions readVSCodeSettings and writeVSCodeSettings are exported from the module
- FileSystemUtils abstraction is used for all file operations
- JSON.parse calls are wrapped in try-catch blocks with error handling
- Input validation logic is present after JSON.parse to check parsed objects against expected schema

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep-based verification commands MUST pass before accepting changes to src/vscode/settings.ts. Code review MUST ensure FileSystemUtils abstraction is used for all file operations and that no direct fs module usage occurs outside approved modules. Static analysis MUST flag violations.
</enforcement>