# Relative Path Module Organization for First-Party Code: Runtime Built Modules Imported Bare Specifiers

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase spans multiple application layers including agents, MCP propagation utilities, and CLI handlers, each requiring access to shared functionality
- First-party modules for file system operations, type definitions, path utilities, and core business logic are organized in dedicated directories (core/, types/, paths/, lib/)
- Modules across different layers (src/agents/, src/mcp/, src/cli/) need explicit dependency relationships to shared utilities without introducing circular dependencies
- The project uses relative path imports to establish clear module boundaries and ownership, making dependency graphs explicit in import statements

## Problem Statement

In a multi-layer TypeScript/Node.js application with shared utilities and cross-cutting concerns, teams need a consistent convention for importing first-party modules that makes dependency relationships explicit, prevents circular dependencies, and maintains clear architectural boundaries between layers.

## Decision

1. SHOULD: Runtime built-in modules SHOULD be imported using bare specifiers without path traversal

## Policy Block

- SHOULD Runtime built-in modules SHOULD be imported using bare specifiers without path traversal

In scope:
- All first-party TypeScript/JavaScript modules within the project workspace
- Imports between modules in different directories or application layers
- Shared utilities, type definitions, and core business logic accessed from multiple layers

Out of scope:
- Third-party dependencies installed from package registries
- Runtime built-in modules provided by the execution environment
- Modules within the same directory using same-level relative imports

## Rationale

- Relative path imports make dependency relationships explicit in the source code, enabling developers to understand module coupling by reading import statements
- The directory structure (core/, types/, paths/, lib/) observed across 3 files with high significance (0.89-0.93) establishes architectural boundaries that separate concerns and prevent tight coupling
- Explicit path traversal (../) enforces awareness of layer boundaries and discourages inappropriate cross-layer dependencies
- This convention aligns with standard Node.js module resolution while maintaining architectural visibility without requiring additional tooling for path aliases

## Consequences

Positive:
- Dependency relationships are immediately visible in import statements without consulting external configuration
- Architectural boundaries between layers (agents, utilities, types) are enforced through explicit path traversal
- Module ownership and responsibility are clear from directory organization
- No additional build-time configuration or path alias resolution required

Negative:
- Refactoring module locations requires updating all relative import paths in dependent modules
- Deep directory nesting results in verbose import paths with multiple upward traversals (../../..)
- IDE refactoring tools may not always correctly update relative paths during file moves
- New developers must learn the directory structure to construct correct import paths

## Alternatives

- Path alias configuration using module resolution mapping to create short, absolute-style imports (rejected)
  Rejected because: Evidence shows consistent use of relative paths across all observed files; no path alias configuration detected in the pattern
  When valid: When import path verbosity becomes a significant maintenance burden or when the team prioritizes refactoring ease over explicit dependency visibility
- Monorepo workspace packages with explicit package.json dependencies between internal modules (rejected)
  Rejected because: The observed pattern uses relative imports within a single source tree rather than separate workspace packages
  When valid: When modules need independent versioning, separate build artifacts, or when enforcing strict API boundaries between major subsystems
- Barrel exports (index files) to simplify import paths and hide internal module structure (deferred)
  Rejected because: Evidence does not show barrel export usage; imports target specific module files directly
  When valid: When public API surface needs to be distinguished from internal implementation details within a module directory

## Risks

- Large-scale directory restructuring becomes expensive as the number of modules and import relationships grows
  Mitigation: Establish directory structure early and use automated refactoring tools that understand TypeScript module resolution; consider introducing path aliases if refactoring pain exceeds acceptable thresholds
  Owner: engineering team
- Developers may create circular dependencies that are not immediately obvious from relative import paths alone
  Mitigation: Implement automated circular dependency detection in continuous integration; enforce layered architecture where lower-level modules never import from higher-level modules
  Owner: engineering team
- Inconsistent application of the convention leads to mixed import styles that reduce architectural clarity
  Mitigation: Document the convention explicitly; use linting rules to enforce relative path usage for first-party imports; conduct code review with focus on import path consistency
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
- When creating new modules, place them in the appropriate directory based on their architectural role: shared utilities in core/, type definitions in types/, domain-specific paths in paths/, and reusable business logic in lib/
- When importing from a module in a different directory, construct the relative path by counting directory levels upward to the common ancestor, then downward to the target module
- Use IDE features for automatic import path generation to reduce manual path construction errors; verify generated paths follow the relative convention rather than introducing aliases

## Continuation Context


Verify commands:
- Discover the project's module resolution configuration and verify no path alias mappings override relative import behavior for first-party modules
- Discover and execute the project's static analysis tooling to detect circular dependencies in the module graph
- Discover and execute the project's linting configuration to verify enforcement of relative path imports for first-party modules

Accept when:
- All first-party module imports use relative paths with explicit directory traversal
- No circular dependencies exist in the module dependency graph
- Directory structure clearly separates concerns with dedicated folders for utilities, types, and business logic

## Enforcement

- Verified by: Static analysis tools that parse import statements and verify relative path usage
- Verified by: Circular dependency detection in continuous integration pipeline
- Verified by: Code review checklist items for import path consistency
- Violation handling: Automated linting failures block pull request merges when non-relative imports are detected for first-party modules
- Violation handling: Code review feedback requests correction of import paths that violate the convention
- Violation handling: Circular dependency detection failures halt the build and require immediate resolution
- Exception process: Exceptions for path aliases or alternative import strategies require architectural review and documentation of rationale
- Exception process: Temporary violations during large refactoring efforts must be tracked with technical debt tickets and resolution timelines
- Exception process: Approved exceptions must be documented in code comments explaining why the standard convention does not apply