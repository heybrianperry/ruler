# Standardize Node.js Core Module Usage for File System and Path Operations: Asynchronous File Operations

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all TypeScript modules performing file system operations, configuration loading, or path manipulation.

## Context

- The codebase consists of 16 TypeScript modules performing file system operations, configuration management, and agent coordination across CLI handlers, core utilities, and agent implementations.
- Node.js core modules ('fs', 'fs/promises', 'path', 'os') are consistently imported across configuration loaders (UnifiedConfigLoader, ConfigLoader), file system utilities (FileSystemUtils, GitignoreUtils), agent implementations (FirebenderAgent, GeminiCliAgent, AgentsMdAgent), and integration test harnesses.
- The project requires cross-platform path resolution, environment variable access (XDG_CONFIG_HOME), and both synchronous and asynchronous file operations for reading/writing JSON, TOML, and markdown files.
- Third-party parsing libraries (@iarna/toml, zod) are used alongside core modules for configuration validation and deserialization, establishing a pattern of core module usage for I/O with specialized libraries for data transformation.
- The detection spans agent configuration management, skills propagation, MCP server integration, VSCode settings manipulation, and CLI command handlers, indicating a system-wide architectural commitment to Node.js core APIs.

## Problem Statement

The system requires a consistent, maintainable approach to file system operations, path manipulation, and environment configuration access across diverse modules including agents, CLI handlers, configuration loaders, and utility libraries, while ensuring cross-platform compatibility and supporting both synchronous and asynchronous I/O patterns.

## Decision

1. SHOULD: Asynchronous file operations SHOULD prefer 'fs/promises' API over callback-based 'fs' methods for improved readability and error handling.

## Policy Block

- SHOULD Asynchronous file operations SHOULD prefer 'fs/promises' API over callback-based 'fs' methods for improved readability and error handling.

In scope:
- All TypeScript modules in src/agents/, src/core/, src/cli/, src/paths/, src/vscode/
- Configuration loaders (ConfigLoader, UnifiedConfigLoader)
- File system utilities (FileSystemUtils, GitignoreUtils, SkillsUtils)
- Agent implementations (FirebenderAgent, GeminiCliAgent, AgentsMdAgent, AbstractAgent)
- CLI command handlers (handlers.ts)
- Integration test harnesses requiring file system setup
- MCP configuration readers and writers
- VSCode settings manipulation modules

Out of scope:
- Browser-based or web platform code (if any exists)
- External libraries' internal file operations
- Build scripts or tooling configuration files (webpack, rollup, etc.)
- Documentation generation tools

Exceptions:
- EXC-001: A specialized file system operation requires features not available in Node.js core modules (e.g., file watching with advanced filtering, atomic writes with specific guarantees)

## Rationale

- Evidence shows 16 files consistently importing 'fs', 'fs/promises', 'path', and 'os' across agents, configuration loaders, utilities, and CLI handlers, demonstrating an established architectural pattern with 90.48% confidence.
- Node.js core modules provide zero-dependency, stable APIs for file system operations that are maintained by the Node.js project and guaranteed to be available in all Node.js environments without additional installation.
- Using core modules reduces dependency bloat, eliminates version conflicts with third-party file system libraries, and ensures long-term maintainability as core APIs have stronger backward compatibility guarantees.
- The pattern enables consistent error handling, path resolution, and cross-platform behavior across Windows, macOS, and Linux environments through Node.js's platform abstraction layer.

## Consequences

Positive:
- Zero external dependencies for file system operations reduces package size, installation time, and potential security vulnerabilities from third-party libraries.
- Consistent API surface across all modules simplifies onboarding, code review, and maintenance by establishing predictable patterns for file I/O.
- Cross-platform compatibility is guaranteed by Node.js's platform abstraction, eliminating platform-specific bugs in path handling and file operations.
- Performance is optimized as core modules are implemented in native code and do not incur JavaScript wrapper overhead from third-party abstractions.

Negative:
- Lower-level APIs require more boilerplate code compared to higher-level abstractions provided by libraries like fs-extra (e.g., ensureDir, copy with options).
- Developers must manually implement common patterns like recursive directory creation, atomic writes, or safe file replacement that some libraries provide out-of-box.
- Error handling requires explicit try-catch blocks and error code checking rather than simplified error handling provided by some abstraction libraries.
- Migration to alternative file system abstractions (e.g., for virtual file systems in testing) requires more refactoring than if an abstraction layer were used.

## Alternatives

- Use fs-extra library for enhanced file system operations with utility methods like ensureDir, copy, move, and remove (rejected)
  Rejected because: Adds external dependency for features that can be implemented with core modules; evidence shows the codebase already successfully implements required patterns using core APIs across 16 files without fs-extra
  When valid: Valid for projects requiring extensive file system manipulation with complex recursive operations where development velocity is prioritized over minimal dependencies
- Create internal abstraction layer wrapping core modules to provide simplified, project-specific file system API (deferred)
  Rejected because: Not rejected but deferred; FileSystemUtils.ts already provides some abstraction (findRulerDir, writeGeneratedFile, backupFile); could be expanded if patterns become repetitive
  When valid: Valid if analysis shows significant code duplication in file operations across modules, justifying centralized abstraction
- Use graceful-fs for improved error handling and resilience against EMFILE errors on Windows (rejected)
  Rejected because: No evidence in the IR data of EMFILE errors or file descriptor exhaustion issues; premature optimization without demonstrated need
  When valid: Valid if production monitoring reveals file descriptor exhaustion or concurrent file access issues on Windows platforms

## Risks

- Developers unfamiliar with Node.js core APIs may introduce platform-specific bugs by incorrectly using path.join vs path.resolve or hardcoding path separators
  Mitigation: Provide code examples in FileSystemUtils.ts, enforce linting rules against hardcoded path separators, and include cross-platform path handling in code review checklist
  Owner: Engineering team, enforced through code review and linting
- Boilerplate code for common operations (recursive directory creation, safe file writes) may lead to inconsistent implementations across modules
  Mitigation: Expand FileSystemUtils.ts to provide reusable utility functions for common patterns; existing functions like ensureDirExists, backupFile, writeGeneratedFile serve as templates
  Owner: Core utilities maintainer
- Synchronous fs operations in CLI handlers or agents may block event loop and degrade performance under concurrent operations
  Mitigation: Prefer fs/promises for all I/O operations except initialization code; add performance monitoring to detect event loop blocking; document when synchronous operations are acceptable
  Owner: Engineering team, monitored through performance testing

## Implementation Notes

- Reference FileSystemUtils.ts (src/core/FileSystemUtils.ts) as the canonical example of core module usage patterns including error handling, directory traversal, and backup operations.
- For asynchronous operations, import from 'fs/promises' and use async/await syntax; for synchronous operations (e.g., CLI initialization), import from 'fs' and use *Sync methods explicitly.
- Always use path.join() for combining path segments and path.resolve() for absolute path resolution; never concatenate paths with string operations or hardcoded separators.
- When accessing environment variables for configuration paths (XDG_CONFIG_HOME, HOME), provide fallback values using os.homedir() for cross-platform compatibility as demonstrated in ConfigLoader.ts and FileSystemUtils.ts.
- Wrap JSON.parse() calls in try-catch blocks with descriptive error messages including file path context, as shown in UnifiedConfigLoader.ts, mcp.ts, and agent implementations.

## Continuation Context


Verify commands:
- grep -r "from ['\"]fs['\"]" src/ tests/ --include='*.ts' | wc -l
- grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l
- grep -r "from ['\"]fs-extra['\"]" src/ --include='*.ts' && echo 'FAIL: fs-extra found' || echo 'PASS: no fs-extra'
- grep -r "process\.env" src/ --include='*.ts' | grep -E '(XDG_CONFIG_HOME|HOME)' | wc -l

Accept when:
- All TypeScript modules performing file I/O import 'fs' or 'fs/promises' and 'path' from Node.js core modules, with no imports of third-party file system libraries like fs-extra
- Path operations use path.join() or path.resolve() methods rather than string concatenation, verified by absence of hardcoded path separators ('/' or '\\') in path construction
- Environment variable access for configuration uses process.env with appropriate fallbacks to os.homedir() for cross-platform compatibility
- JSON parsing of file contents uses native JSON.parse() after reading file content, with try-catch error handling including file path context

## Enforcement

- Verified by: Static analysis via grep commands checking for core module imports and absence of third-party file system libraries
- Verified by: Code review checklist verifying path operations use path module methods and avoid string concatenation
- Verified by: Integration tests in tests/integration/ that exercise file system operations across platforms
- Verified by: Linting rules (if configured) to detect hardcoded path separators or discouraged file system patterns
- Violation handling: Code review rejection for new code introducing third-party file system libraries without documented exception approval
- Violation handling: Refactoring task created for existing violations discovered during maintenance
- Violation handling: Build warnings or errors from linting rules detecting hardcoded path separators or incorrect path operations
- Violation handling: Documentation of violation in technical debt register if immediate fix is not feasible
- Exception process: Developer documents specific limitation of Node.js core modules that necessitates third-party library in module comments or ADR amendment
- Exception process: Architecture review evaluates whether limitation can be addressed through internal utility functions in FileSystemUtils.ts
- Exception process: If approved, exception is documented in this ADR's policy_exceptions section with justification and scope limitation
- Exception process: Exception is reviewed quarterly to determine if Node.js core module improvements or internal utilities can eliminate the need