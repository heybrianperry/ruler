# Use Set-Based Visited Directory Tracking for Cycle Prevention in Filesystem Traversal: Filesystem Traversal Operations

These rules are ALWAYS ACTIVE for all recursive directory traversal operations in filesystem utilities, configuration file discovery across XDG_CONFIG_HOME and project directories, and any code path that follows symbolic links during directory enumeration.

### Rules

- **R-FST-001** MUST: Filesystem traversal operations MUST maintain a Set data structure containing visited directory paths to detect cycles.
- **R-FST-002** MUST: Initialize a new Set<string> at the start of each top-level traversal operation and pass it through recursive calls.
- **R-FST-003** MUST: Use fs.realpathSync or fs.promises.realpath to resolve symbolic links before checking the visited set.
- **R-FST-004** MUST: Ensure the visited set contains normalized absolute paths to avoid false negatives from path representation differences.
- **R-FST-005** SHOULD: Wrap real path resolution in try-catch blocks and log errors using console.error while continuing traversal of remaining directories.
- **R-FST-006** SHOULD: Consider exposing the visited set size in debug logs to help diagnose traversal performance issues.
- **R-FST-007** SHOULD: Implement maximum traversal depth as a secondary safety mechanism and monitor memory usage in production.

### Verify

```bash
# Check for visitedDirectories.add pattern in filesystem traversal functions
grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts

# Check for Set initialization with visit-related naming
grep -r 'new Set<string>' src/core/FileSystemUtils.ts | grep -i 'visit'

# Verify visitedDirectories pattern is present and used with .add()
node -e "const fs = require('fs'); const code = fs.readFileSync('src/core/FileSystemUtils.ts', 'utf8'); console.log(code.includes('visitedDirectories') && code.includes('.add(') ? 'PASS' : 'FAIL')"
```

**Accept when:**
- The visitedDirectories.add() pattern is present in filesystem traversal functions
- Real path resolution occurs before checking the visited set
- Traversal functions accept or initialize a Set parameter for tracking visited directories
- All recursive directory operations include cycle detection via visited set
- Symbolic link resolution is wrapped in error handling

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All filesystem traversal operations must be reviewed for compliance with R-FST-001 through R-FST-007 before merge. Integration tests with circular symbolic link fixtures are required to verify termination behavior.
</enforcement>