# Use Set-Based Visited Directory Tracking for Cycle Prevention in Filesystem Traversal: Filesystem Utilities Provide

These rules are ALWAYS ACTIVE for all recursive directory traversal operations in filesystem utilities, configuration file discovery across XDG_CONFIG_HOME and project directories, and any code path that follows symbolic links during directory enumeration.

### Rules

- **R-FS-001** MUST: Initialize a new Set<string> at the start of each top-level traversal operation and pass it through recursive calls to track visited directories.
- **R-FS-002** MUST: Use fs.realpathSync or fs.promises.realpath to resolve symbolic links before checking the visited set to prevent false negatives from path representation differences.
- **R-FS-003** MUST: Ensure the visited set contains normalized absolute paths to avoid false negatives from different path representations of the same directory.
- **R-FS-004** SHOULD: Filesystem utilities SHOULD provide assertion functions like assertNotSymbolicLink and assertContainingDirectoryInsideRoot to validate path safety before traversal.
- **R-FS-005** SHOULD: Wrap real path resolution in try-catch blocks and log errors using console.error while continuing traversal of remaining directories when resolution fails.
- **R-FS-006** MAY: Expose the visited set size in debug logs to help diagnose traversal performance issues and detect pathological cases with extremely deep directory structures.

### Verify

```bash
# Verify visitedDirectories.add pattern is present in filesystem traversal
grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts

# Verify Set initialization for visited tracking
grep -r 'new Set<string>' src/core/FileSystemUtils.ts | grep -i 'visit'

# Verify visitedDirectories pattern with add method
node -e "const fs = require('fs'); const code = fs.readFileSync('src/core/FileSystemUtils.ts', 'utf8'); console.log(code.includes('visitedDirectories') && code.includes('.add(') ? 'PASS' : 'FAIL')"
```

**Accept when:**
- The visitedDirectories.add() pattern is present in filesystem traversal functions
- Real path resolution occurs before checking the visited set
- Traversal functions accept or initialize a Set parameter for tracking visited directories
- Assertion functions like assertNotSymbolicLink are available for path validation
- Error handling wraps real path resolution in try-catch blocks

<enforcement>
Clause Code MUST NOT skip or defer verification. All recursive directory traversal operations MUST implement Set-based cycle detection. Violations block PR merge until remediated.
</enforcement>