# Use Node.js Core Modules for File System Operations in Agent Layer: Code Performing File Import Core Modules

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- Agent implementations require file system access to read configuration files, MCP definitions, and settings from the local environment
- Node.js runtime provides built-in core modules (path, fs, fs/promises) that offer fundamental file system primitives without external dependencies
- The agent layer spans multiple implementations (QwenCodeAgent, RooCodeAgent, AmazonQCliAgent, OpenCodeAgent, AiderAgent, AugmentCodeAgent, AbstractAgent) that share common file I/O requirements
- A custom FileSystemUtils module exists alongside core module usage, suggesting a layered approach where core modules provide primitives and custom utilities provide project-specific abstractions

## Problem Statement

Agent implementations need a consistent, reliable mechanism for file system operations including path manipulation, file reading, and directory traversal. The solution must work across different agent types without introducing external dependencies, while maintaining compatibility with both synchronous and asynchronous I/O patterns required by different agent workflows.

## Decision

1. MUST: Code performing file I/O MUST import core modules using string literals matching the Node.js module resolution convention

## Policy Block

- MUST Code performing file I/O MUST import core modules using string literals matching the Node.js module resolution convention

In scope:
- All agent implementation classes in the agent layer
- VSCode integration code requiring file system access
- Code reading configuration files, MCP definitions, or settings from the local file system
- Modules performing path resolution or directory traversal

Out of scope:
- Browser-based or web-only code where Node.js core modules are unavailable
- Virtual file system abstractions that do not interact with the physical file system
- Test fixtures or mocks that simulate file system behavior without actual I/O

## Rationale

- Node.js core modules are built into the runtime, eliminating external dependency overhead and version conflicts for fundamental file system operations
- The pattern appears across 8 files with significance scores of 0.90-0.93, indicating an established architectural convention rather than isolated usage
- Core modules provide both synchronous and asynchronous APIs, supporting different agent workflow requirements without introducing multiple competing libraries
- Using runtime-provided primitives ensures long-term stability and compatibility with Node.js LTS versions

## Consequences

Positive:
- Zero external dependencies for file system operations reduces bundle size and eliminates supply chain risk for core functionality
- Consistent API surface across all agent implementations simplifies maintenance and onboarding
- Native performance characteristics without abstraction overhead for I/O-intensive agent operations
- Cross-platform path handling via the path module prevents Windows/Unix compatibility issues

Negative:
- Core module APIs are lower-level than some third-party alternatives, requiring more boilerplate for complex operations
- Error handling patterns differ between callback-based fs and promise-based fs/promises, requiring careful API selection
- No built-in retry logic, file watching, or advanced features available in specialized libraries
- Developers must understand Node.js-specific conventions rather than framework-agnostic abstractions

## Alternatives

- Adopt a third-party file system library with enhanced features and unified API (rejected)
  Rejected because: Introduces external dependency for functionality already provided by the runtime; adds supply chain risk and version management overhead for core primitives
  When valid: When advanced features like atomic writes, file watching, or glob patterns are required across many modules
- Build a complete custom file system abstraction layer wrapping all I/O operations (rejected)
  Rejected because: Over-abstraction for current needs; evidence shows core modules used directly alongside targeted utilities, not a complete wrapper
  When valid: When file system operations need to be swappable (e.g., supporting virtual file systems or remote storage backends)
- Use only synchronous fs APIs to simplify control flow (rejected)
  Rejected because: Blocks event loop during I/O operations; evidence shows fs/promises adoption indicating async patterns are required
  When valid: In CLI startup code or build scripts where blocking behavior is acceptable

## Risks

- Mixing callback-based fs and promise-based fs/promises APIs inconsistently across modules creates maintenance burden and error-prone code
  Mitigation: Establish convention preferring fs/promises for new code; document when synchronous APIs are acceptable
  Owner: engineering team
- Direct use of core modules without error handling wrappers may lead to unhandled ENOENT, EACCES, or permission errors
  Mitigation: FileSystemUtils module should provide error handling patterns; code review should verify error handling for all I/O operations
  Owner: engineering team
- Path manipulation errors (e.g., incorrect relative path resolution) may cause file not found errors in different execution contexts
  Mitigation: Enforce use of path module for all path operations; test agents in different working directory contexts
  Owner: engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When reading JSON configuration files, wrap file read operations and JSON parsing in try-catch blocks to handle both I/O errors and parse errors separately
- Use path.resolve() or path.join() with appropriate base paths rather than string concatenation to construct file paths, ensuring cross-platform compatibility
- For agent initialization code that reads configuration, prefer fs/promises with async/await to avoid blocking the event loop during startup

## Continuation Context


Verify commands:
- Discover the project's static analysis configuration and execute the import validation rules to confirm core module usage
- Locate the project's test suite and run file system operation tests to verify core module integration
- Inspect agent implementation files to confirm imports match the required core module pattern

Accept when:
- All agent implementation files import file system functionality exclusively from Node.js core modules (path, fs, fs/promises)
- Static analysis confirms no third-party file system libraries are imported in agent layer code
- File system operation tests pass using the core module APIs

## Enforcement

- Verified by: Static analysis tooling scanning import statements in agent layer modules
- Verified by: Code review checklist verifying core module usage for new file I/O operations
- Verified by: Automated dependency analysis flagging third-party file system libraries in agent dependencies
- Violation handling: CI pipeline fails if third-party file system libraries are detected in agent layer dependencies
- Violation handling: Code review blocks merge if file I/O operations bypass core modules without documented exception
- Violation handling: Architecture review required for any proposal to introduce alternative file system libraries
- Exception process: Document specific limitation of core modules that requires alternative library
- Exception process: Propose exception in architecture review with scope limited to specific use case
- Exception process: Update policy scope to clarify when exception applies