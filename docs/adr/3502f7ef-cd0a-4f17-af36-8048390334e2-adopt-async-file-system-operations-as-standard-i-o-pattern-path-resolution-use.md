# Adopt Async File System Operations as Standard I/O Pattern: Path Resolution Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all file system operations across agents, configuration loaders, and utility modules.

## Context

- The codebase performs extensive file system operations across agents, configuration loaders, and utility modules, requiring consistent I/O patterns for reading and writing JSON, TOML, and markdown files
- Multiple modules (FirebenderAgent, GeminiCliAgent, UnifiedConfigLoader, ConfigLoader, SkillsProcessor, FileSystemUtils) independently implement file operations with varying patterns of synchronous and asynchronous access
- Configuration management requires atomic read-modify-write operations across multiple file formats (JSON for MCP configs, TOML for agent configs, markdown for skills) with error handling and backup capabilities
- The system must coordinate file operations across nested directory structures, including .ruler directories, skills hierarchies, and XDG-compliant config paths, while maintaining consistency during concurrent operations

## Problem Statement

Without a standardized approach to file system operations, the codebase exhibits inconsistent concurrency patterns, error handling, and I/O coordination across modules, leading to potential race conditions, incomplete error recovery, and maintenance complexity when extending file-based features.

## Decision

1. MUST: Path resolution MUST use 'path' module functions (join, resolve, dirname) to ensure cross-platform compatibility

## Policy Block

- MUST Path resolution MUST use 'path' module functions (join, resolve, dirname) to ensure cross-platform compatibility

In scope:
- All agent implementations (FirebenderAgent, GeminiCliAgent, AgentsMdAgent, AbstractAgent)
- Configuration loaders (UnifiedConfigLoader, ConfigLoader)
- File system utilities (FileSystemUtils, SkillsUtils, GitignoreUtils)
- MCP and VSCode settings management (mcp.ts, settings.ts)
- CLI handlers performing file operations (handlers.ts, revert.ts)

Out of scope:
- In-memory data structures and business logic that do not interact with the file system
- Network I/O operations (HTTP requests, API calls)
- Database operations if introduced in future

Exceptions:
- EXC-001: Synchronous file operations are required during module initialization or CLI startup where async context is not available
- EXC-002: Performance-critical hot paths require synchronous reads after benchmarking demonstrates unacceptable latency with async operations

## Rationale

- Evidence shows 16 files implementing async file operations (loadUnifiedConfig, applyRulerConfig, readVSCodeSettings, writeVSCodeSettings, discoverSkills, propagateSkills, updateGitignore, revertAllAgentConfigs) with consistent patterns of promise-based coordination
- The paradigm.concurrency_model facet detection across modules demonstrates architectural commitment to non-blocking I/O, enabling concurrent agent configuration, skills propagation, and MCP server management without blocking the event loop
- Centralized utility functions (FileSystemUtils, SkillsUtils) with public contracts (findRulerDir, readMarkdownFiles, writeGeneratedFile, backupFile, walkSkillsTree, copySkillsDirectory) indicate intentional abstraction of file system complexity
- JSON.parse() usage with error handling in 6 modules (FirebenderAgent, GeminiCliAgent, UnifiedConfigLoader, mcp.ts, settings.ts, ConfigLoader) establishes a consistent pattern for configuration deserialization with validation

## Consequences

Positive:
- Non-blocking I/O enables concurrent execution of agent configuration, skills propagation, and MCP setup operations, improving CLI responsiveness and throughput
- Centralized file system utilities reduce code duplication and provide consistent error handling, backup, and atomic write patterns across all modules
- Async function signatures enable proper error propagation through promise chains, simplifying error handling in CLI handlers and agent implementations
- Path module usage ensures cross-platform compatibility for Windows, macOS, and Linux environments

Negative:
- Async/await patterns increase code complexity compared to synchronous operations, requiring developers to understand promise semantics and error handling
- Potential for race conditions when multiple operations modify the same configuration files concurrently without explicit locking or coordination
- Memory overhead from promise allocation and async context switching may impact performance in scenarios with thousands of small file operations
- Debugging async stack traces is more complex than synchronous code, potentially increasing troubleshooting time

## Alternatives

- Use synchronous fs module functions (readFileSync, writeFileSync) throughout the codebase (rejected)
  Rejected because: Synchronous operations block the event loop, preventing concurrent agent operations and degrading CLI responsiveness during multi-file operations like skills propagation
  When valid: Only valid for one-time initialization code or CLI startup where async context is unavailable
- Implement a file system abstraction layer with explicit locking and transaction support (deferred)
  Rejected because: Current evidence does not show race condition issues requiring explicit locking; added complexity may not be justified without demonstrated need
  When valid: Valid if concurrent modification bugs emerge or if multi-process coordination becomes necessary
- Use a third-party file system library (fs-extra, graceful-fs) for enhanced functionality (rejected)
  Rejected because: Native fs/promises and fs modules provide sufficient functionality for current requirements; additional dependency increases bundle size and maintenance burden
  When valid: Valid if advanced features like atomic operations, recursive copy with filtering, or enhanced error recovery become requirements

## Risks

- Race conditions may occur when multiple async operations modify the same configuration file concurrently (e.g., parallel agent config updates)
  Mitigation: Implement file-level locking or serialize related operations through a coordination layer; add integration tests for concurrent scenarios
  Owner: Engineering team
- Incomplete error handling in async operations may leave configuration files in inconsistent states after partial writes or parse failures
  Mitigation: Enforce backup-before-write pattern in all configuration updates; implement rollback logic in error handlers; add verification tests
  Owner: Engineering team
- Memory pressure from concurrent file operations on large skills directories or deeply nested .ruler structures may degrade performance
  Mitigation: Implement streaming or batched processing for large directory traversals; add performance benchmarks and monitoring
  Owner: Engineering team

## Implementation Notes

- Use 'fs/promises' import for new code; wrap callback-based 'fs' functions in util.promisify() for legacy compatibility
- Implement backupFile() pattern from FileSystemUtils before modifying existing configuration files; store backups with .backup extension
- Use try-catch blocks around JSON.parse() and file operations; provide meaningful error messages including file paths and operation context
- For recursive operations (walkSkillsTree, findRulerDir), use async generators or batched processing to avoid loading entire directory trees into memory
- Export file system functions as public contracts in utility modules; avoid inline file operations in agent or handler code

## Continuation Context


Verify commands:
- grep -r "readFileSync\|writeFileSync" src/ --include="*.ts" | grep -v "test" | wc -l
- grep -r "from 'fs/promises'\|from \"fs/promises\"" src/ --include="*.ts" | wc -l
- grep -r "JSON.parse" src/ --include="*.ts" | grep -v "try" -A 2 -B 2 | grep -c "catch"
- grep -r "async.*function\|async .*=>" src/ --include="*.ts" | grep -E "(read|write|load|save|update|propagate|discover)" | wc -l

Accept when:
- Synchronous file operations (readFileSync, writeFileSync) are absent or minimal (<5 occurrences) in src/ directory excluding tests
- At least 80% of file operation functions use async/await or return Promises, verified by grep pattern matching
- All JSON.parse() calls are wrapped in try-catch blocks or error handling logic, verified by code review or static analysis
- Integration tests pass for concurrent agent configuration and skills propagation scenarios without race conditions or data corruption

## Enforcement

- Verified by: Code review checklist requiring async file operations for all new file I/O code
- Verified by: Static analysis rules detecting synchronous fs module usage outside approved exceptions
- Verified by: Integration test suite covering concurrent file operations and error scenarios
- Violation handling: Pull requests introducing synchronous file operations are flagged for review and justification
- Violation handling: CI pipeline fails if grep-based verification commands detect policy violations above threshold
- Violation handling: Existing violations are tracked in technical debt backlog with prioritization based on module criticality
- Exception process: Developer documents synchronous operation requirement with inline comment explaining why async is not feasible
- Exception process: Tech lead reviews exception request and approves if justified by initialization constraints or performance data
- Exception process: Approved exceptions are recorded in ADR amendments with rationale and scope limitations