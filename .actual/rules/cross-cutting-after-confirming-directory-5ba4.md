# Use Set-Based Visited Directory Tracking for Cycle Prevention in Filesystem Traversal: After Confirming Directory

These rules are ALWAYS ACTIVE for all recursive directory traversal operations in filesystem utilities, configuration file discovery across XDG_CONFIG_HOME and project directories, and any code path that follows symbolic links during directory enumeration.

### Rules

- **R-CYCLE-001** MUST: After confirming a directory has not been visited, its real path MUST be added to the visited set using the add() method.

### Verify

```bash
# Check for visitedDirectories.add pattern in FileSystemUtils
grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts

# Check for Set initialization with visit-related naming
grep -r 'new Set<string>' src/core/FileSystemUtils.ts | grep -i 'visit'

# Verify the pattern is present in the codebase
node -e "const fs = require('fs'); const code = fs.readFileSync('src/core/FileSystemUtils.ts', 'utf8'); console.log(code.includes('visitedDirectories') && code.includes('.add(') ? 'PASS' : 'FAIL')"
```

**Accept when:**
- The visitedDirectories.add() pattern is present in filesystem traversal functions
- Real path resolution occurs before checking the visited set
- Traversal functions accept or initialize a Set parameter for tracking visited directories
- All recursive directory operations include cycle detection via visited set tracking

<enforcement>
Clause R-CYCLE-001 verification is mandatory. Code review of all filesystem traversal functions must confirm visited set usage. Static analysis must detect recursive directory operations without cycle detection. Integration tests with circular symbolic link fixtures must verify termination. PRs introducing new traversal code without the visited set pattern must be blocked.
</enforcement>