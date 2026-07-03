# Use Set-Based Visited Directory Tracking for Cycle Prevention in Filesystem Traversal: When Directory Real

These rules are ALWAYS ACTIVE for all recursive directory traversal operations in filesystem utilities, configuration file discovery across XDG_CONFIG_HOME and project directories, and any code path that follows symbolic links during directory enumeration.

### Rules

- **R-CYCLE-001** MUST: When a directory's real path is found in the visited set, the traversal MUST skip that directory and continue without error.
- **R-CYCLE-002** MUST: Initialize a new Set<string> at the start of each top-level traversal operation and pass it through recursive calls.
- **R-CYCLE-003** MUST: Use fs.realpathSync or fs.promises.realpath to resolve symbolic links before checking the visited set.
- **R-CYCLE-004** MUST: Ensure the visited set contains normalized absolute paths to avoid false negatives from path representation differences.
- **R-CYCLE-005** SHOULD: Wrap real path resolution in try-catch blocks and log errors using console.error while continuing traversal of remaining directories.
- **R-CYCLE-006** SHOULD: Consider exposing the visited set size in debug logs to help diagnose traversal performance issues.

### Verify

```bash
# Check for visitedDirectories.add pattern in filesystem traversal functions
grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts

# Check for Set initialization with visit-related naming
grep -r 'new Set<string>' src/core/FileSystemUtils.ts | grep -i 'visit'

# Verify cycle detection pattern is present
node -e "const fs = require('fs'); const code = fs.readFileSync('src/core/FileSystemUtils.ts', 'utf8'); console.log(code.includes('visitedDirectories') && code.includes('.add(') ? 'PASS' : 'FAIL')"
```

**Accept when:**
- The visitedDirectories.add() pattern is present in filesystem traversal functions
- Real path resolution occurs before checking the visited set
- Traversal functions accept or initialize a Set parameter for tracking visited directories
- All recursive directory operations include cycle detection via visited set
- Error handling for real path resolution failures uses try-catch and continues traversal

<enforcement>
Clause Code MUST NOT skip or defer verification of cycle detection patterns in filesystem traversal. All new traversal functions MUST implement visited set tracking. Existing traversal functions without cycle detection MUST be flagged for refactoring. Integration tests with circular symbolic link fixtures MUST verify termination behavior.
</enforcement>