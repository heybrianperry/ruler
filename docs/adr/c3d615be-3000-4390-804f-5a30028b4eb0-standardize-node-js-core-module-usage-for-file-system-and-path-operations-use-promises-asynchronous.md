# Standardize Node.js Core Module Usage for File System and Path Operations: Use Promises Asynchronous

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase consistently imports Node.js core modules ('fs/promises', 'path', 'os') across 23 files spanning test infrastructure, agent implementations, and core utilities
- File system operations require promise-based async patterns for configuration management, MCP server setup, skill propagation, and agent-specific file manipulation
- Path manipulation is fundamental to cross-platform compatibility for resolving project roots, config paths, backup paths, and agent-specific directory structures
- Test harness infrastructure depends on core modules for setup/teardown of temporary project directories and validation of file system state
- Agent implementations (OpenCodeAgent, WarpAgent, ClineAgent, TraeAgent, KiroAgent) and core processors (SubagentsProcessor, SkillsProcessor) require consistent file system abstractions

## Problem Statement

The system requires reliable, cross-platform file system and path operations across diverse components including agent implementations, configuration loaders, test infrastructure, and skill/subagent processors, necessitating a consistent approach to module selection and usage patterns.

## Decision

1. MUST: Use 'fs/promises' for all asynchronous file system operations requiring promise-based APIs

## Policy Block

- MUST Use 'fs/promises' for all asynchronous file system operations requiring promise-based APIs

In scope:
- All agent implementations (OpenCodeAgent, WarpAgent, ClineAgent, TraeAgent, KiroAgent, AbstractAgent)
- Core processors (SubagentsProcessor, SkillsProcessor, UnifiedConfigLoader)
- Utility modules (FileSystemUtils, path-utils, SubagentsUtils)
- Test infrastructure and harness setup/teardown functions
- Configuration loaders and MCP server management

Out of scope:
- Browser-based or web platform code where Node.js core modules are unavailable
- Third-party library internals that manage their own file system abstractions
- Build scripts or tooling that may require synchronous operations for simplicity

## Rationale

- Evidence shows 23 files consistently importing 'fs/promises', 'path', and 'os' across test frameworks, agent implementations, and core utilities, demonstrating established architectural pattern
- Promise-based fs operations align with async/await patterns observed in agent lifecycle methods (applyRulerConfig), configuration loading (loadUnifiedConfig), and test harness operations
- Cross-platform path handling is critical for agent config paths (.gemini/settings.json, AGENTS.md, CLAUDE.md) and temporary directory management in test infrastructure
- Standardization on Node.js core modules reduces external dependencies, improves maintainability, and ensures consistent behavior across the 91.26% confidence pattern detected in 23 files

## Consequences

Positive:
- Consistent async patterns across agent implementations and core processors reduce cognitive load and improve code maintainability
- Cross-platform compatibility guaranteed through standardized path module usage for Windows, macOS, and Linux environments
- Zero external dependencies for core file system operations reduces supply chain risk and bundle size
- Test infrastructure reliability improved through consistent temporary directory management and cleanup patterns

Negative:
- Promise-based APIs require proper error handling and may introduce complexity in simple synchronous use cases
- Developers must understand Node.js core module semantics and cannot rely on higher-level abstractions from third-party libraries
- Migration of existing synchronous fs code to promise-based patterns requires refactoring effort
- Limited to Node.js runtime environment, preventing code reuse in browser or edge runtime contexts

## Alternatives

- Use third-party file system libraries (e.g., fs-extra, graceful-fs) for enhanced functionality (rejected)
  Rejected because: Adds external dependencies and maintenance burden when Node.js core modules provide sufficient functionality for observed use cases across 23 files
  When valid: When advanced features like atomic writes, recursive copy with filtering, or enhanced error recovery are required beyond core module capabilities
- Use synchronous fs methods for simplicity in non-performance-critical paths (rejected)
  Rejected because: Inconsistent with async patterns observed in agent lifecycle methods and configuration loaders; blocks event loop in Node.js runtime
  When valid: In build scripts or CLI initialization code where blocking behavior is acceptable and code simplicity is prioritized
- Abstract file system operations behind custom utility layer (deferred)
  Rejected because: FileSystemUtils module exists but evidence shows direct core module imports remain prevalent; full abstraction not yet adopted
  When valid: When cross-runtime support (Node.js, Deno, Bun) or extensive mocking for tests becomes a requirement

## Risks

- Unhandled promise rejections in fs/promises operations could cause silent failures or unhandled rejection warnings
  Mitigation: Enforce try-catch blocks or .catch() handlers in all async file operations; enable strict unhandled rejection policies in Node.js runtime
  Owner: Engineering team
- Path traversal vulnerabilities if user-supplied paths are not validated before path.join() or path.resolve() operations
  Mitigation: Implement path validation using isPathInsideOrEqual utility (observed in path-utils.ts); sanitize all external path inputs
  Owner: Security team
- Race conditions in concurrent file operations during test execution or parallel agent configuration writes
  Mitigation: Use atomic write patterns (writeAgentsDirectoryAtomic observed in SubagentsProcessor); implement file locking or operation serialization where necessary
  Owner: Engineering team

## Implementation Notes

- Import fs/promises using 'import fs from "fs/promises"' or 'import { readFile, writeFile } from "fs/promises"' for tree-shaking benefits
- Always use path.join() or path.resolve() for constructing file paths; never concatenate strings with '/' or '\\'
- In test infrastructure, ensure cleanup with fs.rm(path, { recursive: true, force: true }) in afterEach/afterAll hooks to prevent test pollution
- For JSON parsing after file reads, wrap JSON.parse() in try-catch blocks to handle malformed configuration files gracefully (pattern observed in security.input_validation evidence)
- Use fs.access() with .then(() => true).catch(() => false) pattern for existence checks rather than deprecated fs.exists()

## Continuation Context


Verify commands:
- grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v "fs/promises" | wc -l | grep -q '^0$'
- grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l
- grep -r "readFileSync\|writeFileSync" src/agents/ src/core/ --include='*.ts' | wc -l | grep -q '^0$'
- npm ls fs-extra graceful-fs 2>&1 | grep -q 'empty' || echo 'Third-party fs libs detected'

Accept when:
- All source files in src/agents/ and src/core/ use 'fs/promises' for async file operations with zero synchronous fs method calls
- All path construction uses path.join() or path.resolve() with no string concatenation for file paths
- No third-party file system libraries (fs-extra, graceful-fs) appear in package.json dependencies for core functionality
- Test infrastructure consistently uses os.tmpdir() for temporary directories and fs.rm() with force option for cleanup

## Enforcement

- Verified by: Static analysis via grep patterns in CI pipeline checking for synchronous fs methods and string-based path concatenation
- Verified by: Code review checklist requiring fs/promises and path module usage verification
- Verified by: Dependency audit in CI rejecting PRs that introduce third-party file system libraries without architectural review
- Violation handling: CI build fails if synchronous fs methods detected in src/agents/ or src/core/ directories
- Violation handling: PR review blocks merge if path operations use string concatenation instead of path module methods
- Violation handling: Automated comment on PR with remediation guidance linking to this ADR and code examples
- Exception process: Document exception rationale in code comment with ADR-AUTO reference and architectural review approval
- Exception process: Add file path to .eslintignore or verification script exclusion list with expiration date
- Exception process: Create follow-up issue for technical debt remediation if exception is temporary workaround