# Use Node.js Core Modules for File System Operations in VSCode Settings Management: Public Contracts Include

These rules are ALWAYS ACTIVE for the VSCode settings management module (src/vscode/settings.ts) and all file system operations related to MCP server configuration persistence.

### Rules

- **R-VSCODE-001** MUST: Public API contracts MUST include VSCodeSettings, AugmentMcpServer types and readVSCodeSettings, writeVSCodeSettings functions.
- **R-VSCODE-002** MUST: All file system operations MUST use Node.js core modules 'fs' and 'path' without external database dependencies.
- **R-VSCODE-003** MUST: Configuration data MUST be persisted in JSON-formatted files suitable for version control and manual editing.
- **R-VSCODE-004** MUST: In-memory caching MUST use Map data structures (existingServerMap) to track server instances and avoid redundant file operations.
- **R-VSCODE-005** MUST: All file system operations MUST be centralized in FileSystemUtils abstraction to enable consistent error handling.
- **R-VSCODE-006** MUST: JSON.parse calls MUST be wrapped in try-catch blocks and validated against expected schema before populating cache.
- **R-VSCODE-007** MUST: Atomic write patterns (write to temp file, then rename) MUST be used to prevent partial writes from corrupting configuration files.
- **R-VSCODE-008** SHOULD: File locking mechanism or atomic write-rename pattern SHOULD be implemented to prevent concurrent write corruption.
- **R-VSCODE-009** SHOULD: Cache invalidation strategy SHOULD be documented and explicit refresh methods provided if external modification is supported.

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

# Verify FileSystemUtils abstraction usage
grep -r "FileSystemUtils" src/vscode/settings.ts
```

**Accept when:**
- Grep commands confirm presence of 'fs' and 'path' core module imports in src/vscode/settings.ts
- JSON.parse usage is detected for deserializing configuration content
- Map data structure usage with .set() method is confirmed for cache layer implementation
- Public API functions readVSCodeSettings and writeVSCodeSettings are exported from the module
- FileSystemUtils abstraction is used for all file operations
- Try-catch blocks wrap JSON.parse calls with error handling
- Atomic write patterns are implemented in file write operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-VSCODE rules are mandatory for src/vscode/settings.ts. CI build fails if verification commands do not detect required patterns. Code review rejects PRs introducing alternative storage mechanisms without architectural discussion.
</enforcement>