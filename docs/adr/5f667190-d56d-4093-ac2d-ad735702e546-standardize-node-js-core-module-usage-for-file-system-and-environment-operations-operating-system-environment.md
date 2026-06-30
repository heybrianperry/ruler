# Standardize Node.js Core Module Usage for File System and Environment Operations: Operating System Environment

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires consistent access to file system operations, path manipulation, and operating system environment variables across multiple modules including src/core/FileSystemUtils.ts and src/cli/handlers.ts
- Configuration sources must be resolved from multiple locations including XDG_CONFIG_HOME environment variable and process environment, requiring standardized environment access patterns
- File system operations involve security-sensitive operations such as symbolic link detection, directory traversal validation, and root containment checks that require reliable core module APIs
- The project implements a .ruler directory structure with generated files that must be detected and managed through consistent file system utilities

## Problem Statement

Without standardized usage of Node.js core modules for file system and environment operations, the codebase risks inconsistent behavior across different runtime contexts, security vulnerabilities from varied path handling approaches, and maintenance complexity from mixing different APIs for equivalent operations.

## Decision

1. MUST: Operating system environment access MUST use the 'os' core module for OS-specific information and the 'process.env' object for environment variable access

## Policy Block

- MUST Operating system environment access MUST use the 'os' core module for OS-specific information and the 'process.env' object for environment variable access

In scope:
- All TypeScript modules in src/core/ that perform file system operations
- All TypeScript modules in src/cli/ that access environment variables or file system
- Utility modules that provide path resolution, directory traversal, or configuration loading
- Functions that read or write markdown files, generated files, or configuration files

Out of scope:
- Third-party library code that has its own file system abstractions
- Test fixtures or mock implementations that simulate file system behavior
- Browser-compatible modules that cannot use Node.js core modules

## Rationale

- The evidence shows consistent usage of 'fs', 'path', and 'os' core modules across src/core/FileSystemUtils.ts and src/cli/handlers.ts, indicating an established pattern of relying on Node.js standard library for runtime environment operations
- Security-sensitive operations such as symbolic link detection (isSymbolicLink, assertNotSymbolicLink) and containment validation (assertContainingDirectoryInsideRoot) require the reliability and security guarantees provided by Node.js core modules
- Configuration resolution from process.env.XDG_CONFIG_HOME demonstrates a need for standardized environment variable access patterns that are consistent across the codebase
- The pattern supports maintainability by avoiding dependency on external file system libraries and ensuring compatibility with Node.js runtime guarantees

## Consequences

Positive:
- Consistent file system behavior across all modules reduces debugging complexity and ensures predictable path handling on different operating systems
- Security vulnerabilities related to path traversal and symbolic link attacks are mitigated through standardized validation patterns using core module APIs
- Zero external dependencies for file system operations reduces bundle size and eliminates supply chain risks from third-party file system libraries
- Configuration resolution follows XDG Base Directory specification conventions, improving compatibility with Linux/Unix user environments

Negative:
- Direct use of Node.js core modules creates tight coupling to the Node.js runtime, making browser or alternative runtime ports more difficult
- Error handling patterns using console.error are less flexible than structured logging libraries and may complicate log aggregation in production environments
- Developers must understand Node.js core module APIs and their platform-specific behaviors rather than using higher-level abstractions
- Testing requires mocking Node.js core modules, which can be more complex than mocking higher-level file system abstractions

## Alternatives

- Use a cross-platform file system abstraction library such as fs-extra or graceful-fs (rejected)
  Rejected because: Adds external dependency overhead and the codebase evidence shows no need for features beyond Node.js core modules; security-sensitive operations benefit from direct core module usage
  When valid: When advanced features like atomic writes, recursive operations with retries, or enhanced error recovery are required
- Implement a custom file system abstraction layer to decouple from Node.js core modules (rejected)
  Rejected because: Increases maintenance burden and complexity without evidence of multi-runtime requirements; current pattern shows direct core module usage is sufficient
  When valid: When targeting multiple JavaScript runtimes (Deno, Bun, browser) or when file system operations need to be swappable for different backends
- Mix Node.js core modules with third-party libraries based on feature requirements per module (rejected)
  Rejected because: Creates inconsistent patterns across the codebase and increases cognitive load for developers; evidence shows consistent core module usage is already established
  When valid: Never recommended; consistency in file system operations is critical for security and maintainability

## Risks

- Platform-specific behavior differences in Node.js core modules (e.g., path separators, symbolic link handling) may cause unexpected failures on Windows vs Unix systems
  Mitigation: Implement comprehensive cross-platform testing in CI/CD pipeline covering Windows, macOS, and Linux; use path.normalize and path.resolve consistently to handle platform differences
  Owner: Engineering team
- Direct console.error logging may not integrate with structured logging systems or observability platforms in production deployments
  Mitigation: Document logging patterns and provide migration path to structured logging library if production requirements emerge; ensure all error messages include sufficient context for debugging
  Owner: Engineering team
- Symbolic link validation and directory traversal checks may have edge cases that bypass security controls if not thoroughly tested
  Mitigation: Implement comprehensive security test suite covering symbolic link attacks, directory traversal attempts, and boundary condition testing; conduct security review of path validation logic
  Owner: Security and engineering team

## Implementation Notes

- Always use 'fs/promises' for asynchronous file operations to maintain consistent async/await patterns; avoid callback-based 'fs' module APIs except where synchronous operations are explicitly required
- Wrap core module operations in utility functions (e.g., findRulerDir, resolveProjectRootForRulerDir) that add project-specific validation and security checks rather than calling core modules directly throughout the codebase
- When accessing environment variables, check process.env.XDG_CONFIG_HOME first for configuration directories, then fall back to OS-specific defaults using the 'os' module (e.g., os.homedir())
- Implement visited directory tracking (e.g., visitedDirectories.add(realDir)) when traversing directory structures to prevent infinite loops from circular symbolic links

## Continuation Context


Verify commands:
- grep -r "from 'fs'" src/ && grep -r "from 'path'" src/ && grep -r "from 'os'" src/
- grep -r "process\.env\.XDG_CONFIG_HOME" src/
- grep -r "isSymbolicLink\|assertNotSymbolicLink\|assertContainingDirectoryInsideRoot" src/

Accept when:
- All file system operations in src/core/ and src/cli/ use 'fs' or 'fs/promises' core modules with no third-party file system libraries
- All path operations use the 'path' core module and all environment variable access uses process.env
- Configuration directory resolution checks process.env.XDG_CONFIG_HOME and symbolic link validation functions are present in the codebase

## Enforcement

- Verified by: Static analysis tools scanning import statements for 'fs', 'path', and 'os' core modules
- Verified by: Code review checklist requiring verification of Node.js core module usage for file system operations
- Verified by: Automated grep-based verification in CI pipeline checking for required patterns
- Violation handling: CI pipeline fails if third-party file system libraries are introduced without explicit exception approval
- Violation handling: Code review blocks merge if file system operations bypass established utility functions or use non-standard APIs
- Violation handling: Security review required for any changes to symbolic link handling or directory traversal validation logic
- Exception process: Document specific technical requirement that cannot be met with Node.js core modules
- Exception process: Obtain approval from technical lead with justification for exception
- Exception process: Add inline comment explaining exception rationale and link to approval decision