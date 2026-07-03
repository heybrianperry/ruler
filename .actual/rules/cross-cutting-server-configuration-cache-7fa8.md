# Use Node.js Core Modules for File System Operations in VSCode Settings Management: Server Configuration Cache

These rules are ALWAYS ACTIVE for all files in the VSCode settings management module (src/vscode/settings.ts) and related configuration persistence code that handles MCP server definitions and JSON-formatted settings files.

### Rules

- **R-VSCODE-CONFIG-001** MUST: Server configuration cache MUST use Map data structures with server.name as keys and server objects as values (existingServerMap.set pattern).
- **R-VSCODE-CONFIG-002** MUST: All file system operations MUST be centralized in ../core/FileSystemUtils to enable consistent error handling and future storage backend changes.
- **R-VSCODE-CONFIG-003** MUST: JSON.parse calls MUST be wrapped in try-catch blocks and validated against expected schema before populating cache.
- **R-VSCODE-CONFIG-004** MUST: Configuration persistence MUST use Node.js core modules 'fs' and 'path' without external database dependencies.
- **R-VSCODE-CONFIG-005** MUST: Atomic write patterns (write to temp file, then rename) MUST be used to prevent partial writes from corrupting configuration files.
- **R-VSCODE-CONFIG-006** SHOULD: File watching with fs.watch or TTL-based cache invalidation SHOULD be implemented to handle external modification of settings files.
- **R-VSCODE-CONFIG-007** SHOULD: Configuration files SHOULD remain in JSON format to enable human-readable inspection, manual editing, and version control.

### Verify

```bash
# Verify fs and path core module imports are present
grep -r "require.*['\"]fs['\"]"|import.*from.*['\"]fs['\"]\"|require.*['\"]path['\"]"|import.*from.*['\"]path['\"]\"\ src/vscode/settings.ts

# Verify JSON.parse usage for deserialization
grep -r "JSON\\.parse" src/vscode/settings.ts

# Verify Map data structure usage with .set() method
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
- No direct fs module usage exists outside the FileSystemUtils abstraction layer

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands MUST pass before accepting changes to src/vscode/settings.ts. Code review MUST reject PRs introducing alternative storage mechanisms without architectural discussion. CI build MUST fail if verification commands do not detect required patterns.
</enforcement>