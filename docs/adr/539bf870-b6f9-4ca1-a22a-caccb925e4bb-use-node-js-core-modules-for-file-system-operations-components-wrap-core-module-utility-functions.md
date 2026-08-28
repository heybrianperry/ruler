# Use Node.js Core Modules for File System Operations: Components Wrap Core Module Utility Functions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The project is a TypeScript/Node.js application with multiple agent implementations, CLI infrastructure, and core utilities that require file system access for configuration management and state persistence
- Agent implementations (Firebender, Gemini, Codex, Zed) need to read and write configuration files in various formats (JSON, TOML) from the file system
- The CLI layer requires path resolution and file system operations to locate and process configuration files and package metadata
- Core utilities including the revert-engine and MCP path resolution require reliable file system abstractions for state management and path manipulation
- Node.js runtime provides built-in modules for file system operations without requiring external dependencies, reducing the dependency footprint for foundational I/O operations

## Problem Statement

The application requires a consistent, reliable abstraction layer for file system operations across agent implementations, CLI infrastructure, and core utilities. Without a standardized approach, different components might adopt incompatible file I/O patterns, leading to inconsistent error handling, path resolution issues across platforms, and increased maintenance burden. The solution must support synchronous and asynchronous file operations, cross-platform path manipulation, and integration with format-specific parsers for configuration management.

## Decision

1. MAY: Components MAY wrap core module APIs in utility functions to provide domain-specific abstractions, but the underlying implementation MUST use core modules

## Policy Block

- MAY Components MAY wrap core module APIs in utility functions to provide domain-specific abstractions, but the underlying implementation MUST use core modules

In scope:
- Agent implementations that read or write configuration files
- CLI command handlers that access the file system
- Core utilities that perform path resolution or file system queries
- State management components that persist or retrieve data from disk
- Any component that needs to verify file existence, read file metadata, or manipulate file system paths

Out of scope:
- In-memory data structures and operations that do not interact with the file system
- Network I/O operations including HTTP requests or socket communication
- Database operations or other persistence mechanisms that do not use the file system directly
- Browser-based or web platform code that does not have access to Node.js core modules

## Rationale

- Node.js core modules are built-in to the runtime and require no external dependencies, reducing the project's dependency footprint and eliminating version conflicts for foundational I/O operations
- The pattern is observed across 7 files spanning agent implementations, CLI infrastructure, and core utilities with significance scores of 0.90-0.93, indicating an established architectural practice
- Core modules provide cross-platform abstractions that handle operating system differences in path separators, file system semantics, and encoding, reducing platform-specific bugs
- Using standard Node.js APIs ensures compatibility with the broader Node.js ecosystem and allows developers to leverage extensive documentation and community knowledge

## Consequences

Positive:
- Zero external dependencies for foundational file system operations reduces supply chain risk and simplifies dependency management
- Cross-platform path handling is automatic through the path module, reducing platform-specific bugs and conditional logic
- Developers can leverage extensive Node.js documentation and community resources for file system operations
- Consistent API surface across all components simplifies code review and reduces cognitive load when working across different parts of the codebase

Negative:
- Node.js core module APIs are lower-level than some third-party abstractions, requiring more boilerplate for common operations like recursive directory traversal or atomic file writes
- Error handling must be implemented explicitly for each operation rather than being provided by a higher-level abstraction
- Synchronous APIs can block the event loop if used inappropriately, requiring developer discipline to choose async variants where appropriate
- Core modules do not provide advanced features like file watching with debouncing, atomic writes with rollback, or streaming transformations without additional implementation effort

## Alternatives

- Adopt a third-party file system abstraction library that provides higher-level APIs, promise-based interfaces, and additional utilities (rejected)
  Rejected because: Adds an external dependency for foundational operations that are already well-supported by Node.js core modules. The evidence shows direct use of core modules across 7 files, indicating the team has accepted the lower-level API in exchange for zero dependencies.
  When valid: Consider for future refactoring if the codebase grows to require advanced features like atomic writes with rollback, sophisticated file watching, or if the boilerplate for common operations becomes a maintenance burden
- Create a custom file system abstraction layer that wraps core modules and provides domain-specific utilities (deferred)
  Rejected because: Not rejected, but evidence shows direct use of core modules rather than a unified abstraction. This may evolve as the codebase matures.
  When valid: Valid if repeated patterns emerge across multiple components that would benefit from shared utilities, or if testing requires dependency injection of file system operations
- Use different file system libraries for different contexts (e.g., one for CLI, another for agents) (rejected)
  Rejected because: Creates inconsistency across the codebase and increases the learning curve for developers working across different components. The evidence shows consistent use of core modules across all layers.
  When valid: Never valid within this project architecture; consistency is a core architectural principle

## Risks

- Developers may use synchronous file system APIs inappropriately in hot paths, blocking the event loop and degrading application performance
  Mitigation: Establish code review guidelines that flag synchronous API usage outside of initialization and CLI contexts. Add linting rules to detect synchronous calls in async functions.
  Owner: engineering team
- Inconsistent error handling across components may lead to poor error messages or unhandled exceptions when file operations fail
  Mitigation: Create shared error handling utilities that wrap common file operations and provide consistent error context. Document error handling patterns in developer guidelines.
  Owner: engineering team
- Path manipulation errors may introduce security vulnerabilities such as path traversal attacks if user input is incorporated into file paths without validation
  Mitigation: Always validate and sanitize path inputs before file system operations. Use path module functions to resolve and normalize paths. Implement allowlist-based path validation for user-controlled inputs.
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
- When implementing new file system operations, prefer promise-based async APIs over callback-based APIs for better integration with async/await patterns. The core modules provide both callback and promise-based interfaces.
- For configuration file parsing, separate the file reading operation from the parsing operation. Read the file content as a string or buffer using core modules, then pass to format-specific parsers. This separation improves testability and error handling.
- When constructing file paths, always use the path module's join or resolve functions rather than string concatenation to ensure cross-platform compatibility and proper handling of path separators and relative path resolution.

## Continuation Context


Verify commands:
- Discover the project's static analysis configuration and execute the import analysis tool to verify that file system operations import from core modules
- Locate the project's test suite and execute tests covering file system operations to verify they use core module APIs
- Search the codebase for file system operation patterns and verify they follow the path module usage for path manipulation and core file system module for I/O operations

Accept when:
- Static analysis confirms all file system read, write, and metadata operations use Node.js core modules
- Path manipulation operations consistently use the path module across all components
- No direct string concatenation is used for path construction in file system operations

## Enforcement

- Verified by: Code review process checks for proper use of core modules in file system operations
- Verified by: Static analysis tools scan for import patterns and flag non-core module usage for file I/O
- Verified by: Automated tests verify file system operations use core module APIs
- Violation handling: Code review identifies violations and requests changes before merge
- Violation handling: Static analysis failures block CI pipeline until resolved
- Violation handling: Existing violations are tracked as technical debt and prioritized for refactoring
- Exception process: Developer identifies a legitimate need for alternative file system abstraction
- Exception process: Developer documents the rationale and proposes the exception in a design review
- Exception process: Team lead or architect approves the exception with documented justification
- Exception process: Exception is recorded and reviewed periodically to assess if it should become standard practice