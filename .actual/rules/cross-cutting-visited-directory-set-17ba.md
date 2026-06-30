# Use Set-Based Visited Directory Tracking for Cycle Prevention in Filesystem Traversal: Visited Directory Set

These rules are ALWAYS ACTIVE for all recursive directory traversal operations in filesystem utilities, configuration file discovery across XDG_CONFIG_HOME and project directories, and any code path that follows symbolic links during directory enumeration.

### Rules

- **R-VISITED-001** MUST: Initialize a new `Set<string>` at the start of each top-level traversal operation and pass it through all recursive calls to track visited directories.
- **R-VISITED-002** MUST: Use `fs.realpathSync` or `fs.promises.realpath` to resolve symbolic links before checking the visited set to prevent false negatives from path representation differences.
- **R-VISITED-003** MUST: Ensure the visited set contains normalized absolute paths to avoid false negatives from different path representations of the same directory.
- **R-VISITED-004** MUST: Wrap real path resolution in try-catch blocks and log errors using `console.error` while continuing traversal of remaining directories when real path resolution fails.
- **R-VISITED-005** SHOULD: Expose the visited set size in debug logs to help diagnose traversal performance issues and detect pathological cases with extremely deep directory structures.
- **R-VISITED-006** MAY: Scope the visited directory set to individual traversal operations or share it across related operations depending on use case requirements.

### Verify

```bash
# Check for visitedDirectories.add pattern in filesystem traversal functions
grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts

# Check for Set initialization with visited naming
grep -r 'new Set<string>' src/core/FileSystemUtils.ts | grep -i 'visit'

# Verify visitedDirectories pattern is present and used with .add()
node -e "const fs = require('fs'); const code = fs.readFileSync('src/core/FileSystemUtils.ts', 'utf8'); console.log(code.includes('visitedDirectories') && code.includes('.add(') ? 'PASS' : 'FAIL')"
```

**Accept when:**
- The `visitedDirectories.add()` pattern is present in filesystem traversal functions
- Real path resolution occurs before checking the visited set
- Traversal functions accept or initialize a Set parameter for tracking visited directories
- Try-catch blocks wrap real path resolution calls
- Errors from real path resolution are logged and do not halt traversal

<enforcement>
Clause Code MUST NOT skip or defer verification. All filesystem traversal functions must be reviewed to ensure visited set usage. Static analysis must detect recursive directory operations without cycle detection. Integration tests with circular symbolic link fixtures must verify termination. PRs introducing new traversal code without the visited set pattern MUST be blocked.
</enforcement>