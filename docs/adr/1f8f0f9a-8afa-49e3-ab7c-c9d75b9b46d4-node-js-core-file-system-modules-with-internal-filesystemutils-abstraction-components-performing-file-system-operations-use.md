# Node.js Core File System Modules with Internal FileSystemUtils Abstraction: Components Performing File System Operations Use

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The project implements multiple agent types (FirebenderAgent, ZedAgent, CrushAgent, GeminiCliAgent) and utilities (path resolution, MCP propagation) that require file system operations for reading and writing configuration files.
- Evidence shows consistent use of Node.js core modules (path, fs, fs/promises) across 6 files spanning agent implementations, path utilities, and MCP propagation logic, with significance 0.93.
- An internal FileSystemUtils module exists at src/core/FileSystemUtils and is imported alongside core modules, indicating a deliberate architectural choice to provide a shared abstraction layer for common file system operations.
- The pattern avoids third-party file system libraries (such as fs-extra or graceful-fs), suggesting a preference for minimal external dependencies and reliance on Node.js built-in capabilities.
- File system operations are coupled with JSON parsing (JSON.parse) for configuration management, indicating that file I/O is primarily used for structured data persistence.

## Problem Statement

The project requires a consistent, maintainable approach to file system operations across multiple agent implementations and utilities. Without a standardized pattern, developers might introduce inconsistent error handling, duplicate file system logic, or unnecessary third-party dependencies. The architecture must balance the simplicity of Node.js core modules with the need for shared abstractions that reduce code duplication and enforce consistent behavior across components.

## Decision

1. MUST: All components performing file system operations MUST use Node.js core modules (path, fs, or fs/promises) as the primary file system API.

## Policy Block

- MUST All components performing file system operations MUST use Node.js core modules (path, fs, or fs/promises) as the primary file system API.

In scope:
- All agent implementations requiring file system access for configuration, state persistence, or data storage
- Path utility modules that resolve, validate, or manipulate file system paths
- MCP propagation logic and other utilities that read or write files
- Any component in the src/ directory that performs file I/O operations

Out of scope:
- Browser-based or client-side code that does not have access to Node.js APIs
- Pure computation modules that do not interact with the file system
- Network-only operations that do not involve local file persistence
- Third-party libraries or vendored code that have their own file system abstractions

## Rationale

- Node.js core modules provide stable, well-documented, and performant file system APIs that are maintained as part of the Node.js runtime, eliminating the need for external dependencies in most cases.
- The internal FileSystemUtils abstraction centralizes common file system patterns, reducing code duplication across the 6+ files that perform file operations and providing a single location for error handling improvements or behavioral changes.
- Avoiding third-party file system libraries reduces the dependency surface area, minimizes supply chain risk, and simplifies maintenance by relying on Node.js built-in capabilities that are guaranteed to be present in any Node.js environment.
- The pattern observed across agent implementations (FirebenderAgent, ZedAgent, CrushAgent, GeminiCliAgent) and utilities demonstrates that this approach scales effectively across different component types while maintaining consistency.

## Consequences

Positive:
- Consistent file system operation patterns across all agent implementations and utilities, improving code readability and maintainability.
- Reduced external dependencies and smaller dependency tree, lowering supply chain risk and simplifying dependency management.
- Centralized file system logic in FileSystemUtils enables project-wide improvements to error handling, logging, or retry logic without modifying individual components.
- Developers can rely on standard Node.js documentation and do not need to learn third-party library APIs, reducing onboarding time.

Negative:
- Node.js core file system APIs lack some convenience methods provided by third-party libraries (such as recursive directory creation with error suppression or atomic write operations), requiring manual implementation in FileSystemUtils.
- The FileSystemUtils module becomes a critical dependency that must be carefully maintained; bugs or breaking changes in this module affect all file system operations across the project.
- Developers accustomed to third-party file system libraries may need to adjust to Node.js core module patterns and may initially find them more verbose.
- Advanced file system features (such as file watching with debouncing, glob pattern matching, or cross-platform path normalization edge cases) may require additional implementation effort compared to using mature third-party libraries.

## Alternatives

- Adopt a third-party file system library (such as fs-extra) as the standard file system API across the project (rejected)
  Rejected because: The evidence shows consistent use of Node.js core modules across 6 files with high significance (0.93), indicating that core modules meet the project's needs without requiring additional dependencies. Introducing a third-party library would add dependency overhead without clear architectural benefit given the current usage patterns.
  When valid: Valid if the project encounters repeated need for advanced file system features (such as atomic writes, advanced glob patterns, or recursive operations with complex filtering) that would require significant custom implementation in FileSystemUtils.
- Use Node.js core modules directly without an internal FileSystemUtils abstraction layer (rejected)
  Rejected because: The evidence shows FileSystemUtils is consistently imported alongside core modules across multiple files, indicating that shared abstractions are necessary to avoid code duplication. Removing this layer would require duplicating common patterns (such as error handling, path resolution, or file existence checks) across all agent implementations.
  When valid: Valid for simple, one-off file operations that do not benefit from abstraction or for components that have unique file system requirements not shared with other parts of the codebase.
- Implement a comprehensive file system abstraction that completely wraps Node.js core modules and hides them from consumers (rejected)
  Rejected because: The evidence shows direct imports of both core modules and FileSystemUtils, indicating a hybrid approach where developers use core modules for basic operations and FileSystemUtils for shared patterns. A complete abstraction would add unnecessary indirection for simple operations and increase the maintenance burden of the FileSystemUtils module.
  When valid: Valid if the project needs to support multiple runtime environments (such as Deno or browser with polyfills) or requires comprehensive mocking and testing infrastructure that benefits from a complete abstraction layer.

## Risks

- The FileSystemUtils module becomes a bottleneck for file system operations if it grows too large or accumulates technical debt, affecting all components that depend on it.
  Mitigation: Establish clear guidelines for what belongs in FileSystemUtils (shared, reusable patterns) versus what should remain in individual components (component-specific logic). Regularly review and refactor FileSystemUtils to maintain code quality and prevent it from becoming a catch-all utility module.
  Owner: Engineering team
- Developers unfamiliar with Node.js core file system APIs may introduce bugs related to path handling, encoding, or error handling that would be handled automatically by third-party libraries.
  Mitigation: Document common patterns and gotchas in FileSystemUtils. Provide code review guidelines that specifically check for proper error handling, path normalization, and encoding specification in file system operations. Consider adding linting rules to catch common mistakes.
  Owner: Engineering team
- Cross-platform compatibility issues (Windows vs. Unix path separators, file permissions, case sensitivity) may emerge if not carefully handled in FileSystemUtils abstractions.
  Mitigation: Use the path module consistently for all path operations to ensure cross-platform compatibility. Add integration tests that run on multiple operating systems to catch platform-specific issues early. Document platform-specific considerations in FileSystemUtils.
  Owner: Engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Examine the existing FileSystemUtils module to understand what abstractions are already available before implementing new file system operations. Common patterns such as reading JSON files, ensuring directories exist, or resolving paths relative to project root may already be implemented.
- When adding new methods to FileSystemUtils, ensure they are generic and reusable across multiple components. Component-specific logic should remain in the component, not in the shared utility module.
- Prefer async/await patterns with fs/promises for new code to maintain consistency with modern JavaScript patterns observed in the agent implementations. Use synchronous APIs only when blocking behavior is explicitly required and document the rationale.
- Always specify file encoding explicitly when reading or writing text files to avoid platform-dependent default encoding issues. The evidence shows JSON parsing operations, which require UTF-8 encoding.

## Continuation Context


Verify commands:
- Discover and inspect the project's static analysis configuration to identify any existing rules that verify file system module imports
- Search the codebase for all imports of file system modules and verify they conform to the pattern of using core modules and the internal FileSystemUtils abstraction
- Locate and run the project's test suite to verify that file system operations are properly tested and that FileSystemUtils abstractions are covered by unit tests

Accept when:
- All file system operations in the codebase use Node.js core modules (path, fs, fs/promises) and no third-party file system libraries are present in dependencies
- The internal FileSystemUtils module is consistently imported and used across agent implementations and utilities for shared file system patterns
- Static analysis or linting passes with no violations of file system module import patterns

## Enforcement

- Verified by: Code review process verifies that new file system operations use Node.js core modules and FileSystemUtils appropriately
- Verified by: Static analysis or linting rules detect imports of third-party file system libraries
- Verified by: Automated tests verify that FileSystemUtils abstractions work correctly and that components use them consistently
- Violation handling: Pull requests introducing third-party file system libraries are flagged during code review and require architectural justification
- Violation handling: Code that duplicates file system logic instead of using FileSystemUtils is refactored during code review to use the shared abstraction
- Violation handling: Violations detected by static analysis tools block CI pipeline until resolved
- Exception process: Developers must document specific capability gaps in Node.js core modules that justify introducing a third-party library
- Exception process: Architectural review is required before adding any third-party file system dependency to evaluate alternatives and impact
- Exception process: Approved exceptions are documented in this ADR or in a supplementary decision record with clear scope and rationale