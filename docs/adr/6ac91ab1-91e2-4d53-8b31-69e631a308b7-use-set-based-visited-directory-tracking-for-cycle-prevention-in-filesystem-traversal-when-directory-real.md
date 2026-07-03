# Use Set-Based Visited Directory Tracking for Cycle Prevention in Filesystem Traversal: When Directory Real

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase performs filesystem traversal operations that must handle symbolic links and prevent infinite loops when directories reference each other cyclically
- Configuration discovery requires searching multiple directory hierarchies including XDG_CONFIG_HOME and project-specific paths, where symbolic links may create circular references
- The fs and path modules from Node.js core provide low-level filesystem operations but do not inherently prevent cycle detection during recursive directory traversal
- Error handling for filesystem operations uses console.error for logging, indicating a need for robust failure modes when encountering filesystem anomalies

## Problem Statement

Without explicit cycle detection during recursive filesystem traversal, the system risks infinite loops when encountering symbolic links that create circular directory references, particularly when searching for configuration files across multiple root paths including user-specific and project-specific directories.

## Decision

1. MUST: When a directory's real path is found in the visited set, the traversal MUST skip that directory and continue without error

## Policy Block

- MUST When a directory's real path is found in the visited set, the traversal MUST skip that directory and continue without error

In scope:
- All recursive directory traversal operations in filesystem utilities
- Configuration file discovery across XDG_CONFIG_HOME and project directories
- Operations using findRulerDir and resolveProjectRootForRulerDir functions
- Any code path that follows symbolic links during directory enumeration

Out of scope:
- Single-file read operations that do not traverse directories
- Operations that explicitly disable symbolic link following
- External library filesystem operations not controlled by core utilities
- Write operations that do not require directory traversal

## Rationale

- The IR evidence shows visitedDirectories.add(realDir) in src/core/FileSystemUtils.ts, demonstrating active use of Set-based cycle detection in production code
- The presence of isSymbolicLink and assertNotSymbolicLink functions indicates explicit handling of symbolic link scenarios where cycles are most likely to occur
- Configuration resolution from process.env.XDG_CONFIG_HOME requires traversing user-controlled directory structures where symbolic links are common
- The pattern provides O(1) lookup performance for cycle detection while maintaining minimal memory overhead proportional to directory depth

## Consequences

Positive:
- Prevents infinite loops and stack overflow errors when encountering circular symbolic link references during filesystem traversal
- Provides deterministic termination guarantees for configuration discovery operations across arbitrary directory structures
- Enables safe traversal of user-controlled directories including XDG_CONFIG_HOME without risk of hanging or resource exhaustion
- Set-based implementation offers O(1) cycle detection with minimal memory overhead

Negative:
- Requires maintaining additional state (visited set) for each traversal operation, increasing memory usage proportional to directory depth
- May skip legitimate directories if multiple symbolic links point to the same real path, potentially missing valid configuration files
- Adds complexity to filesystem utility functions that must coordinate visited set initialization and propagation across recursive calls
- Real path resolution for every directory adds filesystem I/O overhead compared to naive traversal

## Alternatives

- Use depth-limited traversal without cycle detection (rejected)
  Rejected because: Depth limits are arbitrary and may either fail to reach valid configuration files or still encounter cycles within the limit, providing no guarantee of termination
  When valid: In controlled environments where directory structure depth is known and symbolic links are prohibited by policy
- Track visited inodes instead of paths (rejected)
  Rejected because: Inode-based tracking is platform-specific and not reliably available across all Node.js filesystem APIs, reducing portability
  When valid: On Unix-like systems where inode stability is guaranteed and cross-platform compatibility is not required
- Disable symbolic link following entirely (rejected)
  Rejected because: Many legitimate configuration setups use symbolic links for environment-specific overrides or shared configuration, breaking common user workflows
  When valid: In high-security environments where symbolic links are explicitly prohibited and all paths must be canonical

## Risks

- Real path resolution may fail on broken symbolic links, causing traversal to abort prematurely
  Mitigation: Wrap real path resolution in try-catch blocks and log errors using console.error while continuing traversal of remaining directories
  Owner: engineering team
- Visited set may grow unbounded in pathological cases with extremely deep directory structures
  Mitigation: Implement maximum traversal depth as a secondary safety mechanism and monitor memory usage in production
  Owner: engineering team
- Race conditions may occur if directory structure changes during traversal, potentially creating cycles after initial checks
  Mitigation: Accept eventual consistency model for filesystem state and rely on visited set to catch cycles even if they form mid-traversal
  Owner: engineering team

## Implementation Notes

- Initialize a new Set<string> at the start of each top-level traversal operation and pass it through recursive calls
- Use fs.realpathSync or fs.promises.realpath to resolve symbolic links before checking the visited set
- Ensure the visited set contains normalized absolute paths to avoid false negatives from path representation differences
- Consider exposing the visited set size in debug logs to help diagnose traversal performance issues

## Continuation Context


Verify commands:
- grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts
- grep -r 'new Set<string>' src/core/FileSystemUtils.ts | grep -i 'visit'
- node -e "const fs = require('fs'); const code = fs.readFileSync('src/core/FileSystemUtils.ts', 'utf8'); console.log(code.includes('visitedDirectories') && code.includes('.add(') ? 'PASS' : 'FAIL')"

Accept when:
- The visitedDirectories.add() pattern is present in filesystem traversal functions
- Real path resolution occurs before checking the visited set
- Traversal functions accept or initialize a Set parameter for tracking visited directories

## Enforcement

- Verified by: Code review of all filesystem traversal functions to ensure visited set usage
- Verified by: Static analysis to detect recursive directory operations without cycle detection
- Verified by: Integration tests with circular symbolic link fixtures to verify termination
- Violation handling: Block PR merge if new traversal code lacks visited set pattern
- Violation handling: Flag existing traversal functions without cycle detection for refactoring
- Violation handling: Require test coverage demonstrating handling of circular symbolic links
- Exception process: Document justification for why cycle detection is not needed in specific case
- Exception process: Obtain approval from two senior engineers familiar with filesystem utilities
- Exception process: Add inline comments explaining the exception and alternative safety mechanisms