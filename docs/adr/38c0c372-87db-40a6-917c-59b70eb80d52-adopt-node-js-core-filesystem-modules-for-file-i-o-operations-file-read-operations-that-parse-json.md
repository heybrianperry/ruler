# Adopt Node.js Core Filesystem Modules for File I/O Operations: File Read Operations That Parse Json

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent implementations, MCP infrastructure, and file-based configuration management components.

## Context

- The project operates in a Node.js runtime environment where agents and MCP (Model Context Protocol) scripts require filesystem access for reading and writing configuration files in JSON format.
- Six files across agent implementations (FirebenderAgent, CrushAgent, ZedAgent) and MCP propagation scripts demonstrate consistent imports of Node.js core modules (path, fs, fs/promises) for file I/O operations.
- An internal FileSystemUtils module exists in the core package, suggesting a deliberate abstraction layer over raw filesystem operations to ensure consistency and error handling across the codebase.
- The pattern shows JSON.parse operations on file contents with error handling (console.warn), indicating configuration management is a primary use case for filesystem access.
- The architecture requires a stable, well-documented filesystem API that works across different operating systems without external dependencies.

## Problem Statement

Agent implementations and MCP infrastructure components need a reliable, cross-platform mechanism for reading and writing configuration files, managing JSON data persistence, and performing path operations. The solution must work in a Node.js runtime without introducing external dependencies for basic file I/O, while maintaining consistency in error handling and API usage across multiple subsystems.

## Decision

1. MUST: File read operations that parse JSON content MUST implement error handling for both file system errors and JSON parsing failures.

## Policy Block

- MUST File read operations that parse JSON content MUST implement error handling for both file system errors and JSON parsing failures.

In scope:
- Agent implementations that read or write configuration files
- MCP propagation scripts that manage server definitions or configuration state
- Utility modules that provide filesystem abstractions or path operations
- Components that parse JSON data from files or persist JSON data to disk

Out of scope:
- Browser-based or web runtime components that do not have access to Node.js APIs
- Components that require advanced filesystem features not provided by Node.js core modules (file watching with advanced filters, atomic operations, etc.)
- Binary file processing that requires specialized libraries for format parsing

## Rationale

- Node.js core modules provide stable, well-documented, and cross-platform filesystem APIs that are maintained as part of the Node.js runtime, eliminating external dependency risk for fundamental I/O operations.
- The evidence shows 6 files with high significance (0.92-0.93) consistently importing these core modules, indicating an established architectural pattern rather than ad-hoc implementation choices.
- The internal FileSystemUtils abstraction layer demonstrates intentional architectural design to wrap core modules with project-specific error handling and conventions, enabling consistent behavior across agent and MCP subsystems.
- Using core modules for basic file I/O reduces bundle size, eliminates version conflicts with third-party filesystem libraries, and ensures compatibility with the Node.js runtime's security and permission model.

## Consequences

Positive:
- Zero external dependencies for basic filesystem operations reduces supply chain risk and simplifies dependency management.
- Cross-platform compatibility is guaranteed by Node.js core module implementations that abstract OS-specific filesystem differences.
- The FileSystemUtils abstraction provides a single point of control for error handling patterns, logging, and filesystem operation conventions.
- Promise-based fs/promises API enables clean async/await patterns in TypeScript agent implementations without callback complexity.

Negative:
- Node.js core filesystem APIs lack advanced features like recursive directory watching with filters, atomic file operations, or high-performance streaming that specialized libraries provide.
- Error handling must be implemented manually for each operation rather than relying on higher-level abstractions that third-party libraries might offer.
- The FileSystemUtils abstraction creates an internal maintenance burden and requires documentation to ensure consistent usage across the team.
- Migration to alternative runtimes (Deno, Bun) may require refactoring if their filesystem APIs diverge from Node.js core module interfaces.

## Alternatives

- Adopt a third-party filesystem library with enhanced features and abstractions over Node.js core modules (rejected)
  Rejected because: The evidence shows direct use of Node.js core modules across 6 files, and the internal FileSystemUtils module already provides necessary abstractions. Introducing a third-party library would add dependency overhead for operations that core modules handle adequately.
  When valid: When the project requires advanced filesystem features like atomic operations, advanced file watching, or cross-runtime compatibility that core modules cannot provide.
- Use callback-based fs module APIs instead of promise-based fs/promises (rejected)
  Rejected because: The codebase is TypeScript with async/await patterns. Promise-based APIs align better with modern asynchronous code and avoid callback nesting complexity.
  When valid: In legacy codebases or performance-critical paths where callback overhead must be minimized, though this is rarely a practical concern.
- Implement all filesystem operations directly without the FileSystemUtils abstraction layer (rejected)
  Rejected because: The evidence shows FileSystemUtils is imported alongside core modules in multiple files, indicating intentional abstraction for consistency. Direct usage would lead to duplicated error handling and inconsistent patterns across agents and MCP scripts.
  When valid: In isolated utility scripts or one-off tools where consistency with the main codebase is not required.

## Risks

- Inconsistent error handling across different components if developers bypass FileSystemUtils and use core modules directly without proper error handling patterns.
  Mitigation: Enforce code review checks for filesystem operations and document FileSystemUtils usage patterns. Consider linting rules to detect direct fs usage outside the abstraction layer.
  Owner: engineering team
- Node.js core module API changes or deprecations in future Node.js versions could require refactoring across multiple files.
  Mitigation: Monitor Node.js release notes for filesystem API changes. The FileSystemUtils abstraction provides a single point to adapt to API changes, limiting refactoring scope.
  Owner: engineering team
- Performance bottlenecks may emerge if FileSystemUtils abstraction adds overhead to high-frequency file operations.
  Mitigation: Profile filesystem operations under realistic load. If overhead is detected, optimize FileSystemUtils or allow direct core module usage in performance-critical paths with documented exceptions.
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
- Locate the FileSystemUtils module in the core package and review its exported functions before implementing new filesystem operations. Use these abstractions for common patterns like reading JSON files, writing configuration data, and path resolution.
- When implementing error handling for filesystem operations, follow the pattern observed in the evidence: catch both filesystem errors and JSON parsing errors separately, and use appropriate logging mechanisms to record failures without crashing the agent or MCP script.
- For new agent implementations or MCP scripts, import fs/promises for asynchronous operations and path for cross-platform path manipulation. Avoid mixing callback-based fs methods with promise-based code to maintain consistency.

## Continuation Context


Verify commands:
- Discover the project's static analysis configuration and execute the linting tool to verify that filesystem imports follow the established pattern of using core modules and FileSystemUtils.
- Locate the project's test suite directory and run filesystem-related unit tests to confirm that FileSystemUtils abstractions work correctly and error handling patterns are consistent.
- Search the codebase for direct imports of Node.js core filesystem modules and verify they are accompanied by proper error handling or routed through FileSystemUtils.

Accept when:
- All agent implementations and MCP scripts import Node.js core filesystem modules (fs, fs/promises, path) for file I/O operations.
- FileSystemUtils module is imported and used for common filesystem patterns in components that perform file operations.
- Error handling is present for both filesystem errors and JSON parsing failures in all file read operations that parse JSON content.

## Enforcement

- Verified by: Code review process checks for filesystem operation patterns and FileSystemUtils usage.
- Verified by: Static analysis and linting rules detect direct filesystem module usage without proper error handling.
- Verified by: Automated tests verify that filesystem operations follow established patterns and handle errors correctly.
- Violation handling: Code review feedback requests refactoring to use FileSystemUtils or add proper error handling.
- Violation handling: Linting failures block merge until filesystem operations are corrected to follow the established pattern.
- Violation handling: Test failures for filesystem operations require fixes before code can be merged.
- Exception process: Document the technical justification for bypassing FileSystemUtils or using alternative filesystem libraries in code comments.
- Exception process: Obtain approval from a senior engineer or architect during code review for exceptions to the standard pattern.
- Exception process: Record approved exceptions in architecture documentation with rationale and scope limitations.