# Use Node.js Core Modules for File System Operations in VSCode Settings Management: Settings Data Serialized

These rules are ALWAYS ACTIVE for all file system operations and configuration persistence in the VSCode settings management module (src/vscode/settings.ts) and related FileSystemUtils abstractions.

### Rules

- **R-VSCODE-SETTINGS-001** MUST: Settings data MUST be serialized and deserialized using JSON.parse for reading configuration content.
- **R-VSCODE-SETTINGS-002** MUST: All file system operations MUST be centralized in FileSystemUtils to enable consistent error handling and future storage backend changes.
- **R-VSCODE-SETTINGS-003** MUST: JSON.parse calls MUST be wrapped in try-catch blocks and validated against expected schema before populating cache.
- **R-VSCODE-SETTINGS-004** MUST: Atomic write patterns (write to temp file, then rename) MUST be used to prevent partial writes from corrupting configuration files.
- **R-VSCODE-SETTINGS-005** SHOULD: In-memory Map-based caching of server instances by name SHOULD be maintained to reduce redundant file I/O operations.
- **R-VSCODE-SETTINGS-006** SHOULD: Cache invalidation strategy SHOULD be documented and explicit refresh methods SHOULD be provided if external modification of settings files is supported.

### Verify

```bash
# Verify fs and path core module imports
grep -r "require.*['\"]fs['\"]\|import.*from.*['\"]fs['\"]" src/vscode/settings.ts

# Verify JSON.parse usage for deserialization
grep -r "JSON\.parse" src/vscode/settings.ts

# Verify Map data structure usage for cache layer
grep -r "existingServerMap\.set\|Map" src/vscode/settings.ts

# Verify public API functions are exported
grep -r "readVSCodeSettings\|writeVSCodeSettings" src/vscode/settings.ts

# Verify FileSystemUtils abstraction is used
grep -r "FileSystemUtils" src/vscode/settings.ts

# Verify try-catch wrapping around JSON.parse
grep -r "try.*JSON\.parse\|catch" src/vscode/settings.ts
```

**Accept when:**
- Grep commands confirm presence of 'fs' and 'path' core module imports in src/vscode/settings.ts
- JSON.parse usage is detected for deserializing configuration content
- Map data structure usage with .set() method is confirmed for cache layer implementation
- Public API functions readVSCodeSettings and writeVSCodeSettings are exported from the module
- FileSystemUtils abstraction is used for all file operations
- JSON.parse calls are wrapped in try-catch blocks with error handling
- Atomic write patterns are implemented for configuration file writes

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST pass before accepting changes to src/vscode/settings.ts or related FileSystemUtils modules. Code review MUST reject PRs introducing alternative storage mechanisms without architectural discussion and ADR amendment.
</enforcement>