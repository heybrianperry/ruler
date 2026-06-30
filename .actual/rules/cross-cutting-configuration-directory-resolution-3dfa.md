# Use Set-Based Visited Directory Tracking for Cycle Prevention in Filesystem Traversal: Configuration Directory Resolution

These rules are ALWAYS ACTIVE for all recursive directory traversal operations in filesystem utilities, configuration file discovery across XDG_CONFIG_HOME and project directories, and any code path that follows symbolic links during directory enumeration.

### Rules

- **R-CYCLE-001** MUST: All recursive directory traversal operations MUST use Set-based visited directory tracking to prevent infinite loops when encountering circular symbolic link references.
- **R-CYCLE-002** MUST: Configuration directory resolution MUST check environment variables like XDG_CONFIG_HOME before falling back to default paths.
- **R-CYCLE-003** MUST: Real path resolution MUST occur before checking the visited set to ensure normalized absolute paths are used for cycle detection.
- **R-CYCLE-004** MUST: Traversal functions MUST accept or initialize a Set<string> parameter for tracking visited directories and pass it through all recursive calls.
- **R-CYCLE-005** SHOULD: Real path resolution failures on broken symbolic links SHOULD be wrapped in try-catch blocks and logged using console.error while continuing traversal of remaining directories.
- **R-CYCLE-006** SHOULD: Visited set size SHOULD be exposed in debug logs to help diagnose traversal performance issues.
- **R-CYCLE-007** MAY: A maximum traversal depth MAY be implemented as a secondary safety mechanism to prevent unbounded growth of the visited set in pathological cases.

### Verify

```bash
# Verify visitedDirectories.add pattern is present in filesystem traversal
grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts

# Verify Set initialization for visited tracking
grep -r 'new Set<string>' src/core/FileSystemUtils.ts | grep -i 'visit'

# Verify visitedDirectories pattern and .add() usage
node -e "const fs = require('fs'); const code = fs.readFileSync('src/core/FileSystemUtils.ts', 'utf8'); console.log(code.includes('visitedDirectories') && code.includes('.add(') ? 'PASS' : 'FAIL')"
```

**Accept when:**
- The visitedDirectories.add() pattern is present in filesystem traversal functions
- Real path resolution occurs before checking the visited set
- Traversal functions accept or initialize a Set parameter for tracking visited directories
- Configuration resolution checks XDG_CONFIG_HOME environment variable before falling back to defaults
- Error handling for real path resolution failures uses try-catch and console.error logging
- Integration tests with circular symbolic link fixtures verify termination

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All filesystem traversal functions must be reviewed for compliance with R-CYCLE-001 through R-CYCLE-007. PRs introducing new traversal code without visited set pattern MUST be blocked. Existing traversal functions lacking cycle detection MUST be flagged for refactoring.
</enforcement>