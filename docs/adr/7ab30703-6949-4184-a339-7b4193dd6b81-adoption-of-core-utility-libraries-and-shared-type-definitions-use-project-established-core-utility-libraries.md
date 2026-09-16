# Adoption of Core Utility Libraries and Shared Type Definitions: Use Project Established Core Utility Libraries

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires standardized mechanisms for file path manipulation and asynchronous file system operations.
- Configuration files are often managed in YAML format, necessitating a consistent parsing solution.
- To maintain consistency and reusability across different modules, type definitions are centralized.
- Core utility functions are needed across various parts of the application, particularly in agent and configuration management.

## Problem Statement

The project needs a consistent and reliable approach for handling file system interactions, parsing YAML configurations, and managing shared type definitions to ensure code clarity, maintainability, and prevent duplication of effort.

## Decision

1. MUST: MUST use the project's established core utility libraries for file path manipulation, asynchronous file system operations, and YAML configuration parsing.

## Policy Block

- MUST MUST use the project's established core utility libraries for file path manipulation, asynchronous file system operations, and YAML configuration parsing.

In scope:
- Modules responsible for configuration loading, file system interactions, subagent utilities, and shared data structures.

Out of scope:
- Modules that do not require file system access, YAML parsing, or shared type definitions.

## Rationale

- Adopting standard core utility libraries reduces boilerplate code and leverages well-tested implementations for common infrastructure tasks.
- Centralizing type definitions in a dedicated module improves code maintainability, reduces type-related errors, and facilitates refactoring.
- Using established libraries for YAML parsing ensures consistent handling of configuration data across the application.
- Explicitly managing dependency versions via lock files prevents unexpected behavior due to transitive dependency updates.

## Consequences

Positive:
- Improved code consistency and readability due to standardized utility usage.
- Reduced development time by reusing existing, well-understood libraries.
- Enhanced type safety and easier refactoring through centralized type definitions.
- More predictable application behavior due to controlled dependency versions.

Negative:
- Potential for increased bundle size if libraries are not tree-shaken effectively.
- Dependency on external library updates and their potential breaking changes.
- Over-reliance on specific library APIs might make future migrations more complex.

## Alternatives

- Implement custom utility functions for file system operations and YAML parsing. (rejected)
  Rejected because: Increases development overhead, introduces potential for bugs, and duplicates effort already solved by mature libraries.
  When valid: For highly specialized, performance-critical operations where existing libraries introduce unacceptable overhead.
- Define types inline within each module. (rejected)
  Rejected because: Leads to type duplication, inconsistencies, and makes global type changes difficult and error-prone.
  When valid: For types strictly local to a single file and not shared.

## Risks

- External library vulnerabilities.
  Mitigation: Regularly audit dependencies and keep them updated within a controlled version range.
  Owner: Engineering Team
- Breaking changes in library updates.
  Mitigation: Pin major versions and thoroughly test updates in a staging environment before deployment.
  Owner: Engineering Team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- New utility functions should be added to existing core utility modules if they align with their purpose, rather than creating new, isolated utility files.
- All new shared types must be defined in the designated shared types module.

## Continuation Context


Verify commands:
- Discover the project's build configuration and run the type-checking command.
- Discover the project's test runner and execute all unit and integration tests.
- Discover the project's dependency management tool and inspect the lock file for resolved versions of core utility libraries.

Accept when:
- Type-checking completes without errors, indicating correct usage of shared types.
- All tests pass, confirming the functionality relying on core utilities.
- The lock file clearly specifies the versions of adopted core utility libraries.

## Enforcement

- Verified by: Automated CI checks for type errors and test failures.
- Verified by: Code reviews ensuring adherence to utility usage and type definition guidelines.
- Violation handling: CI pipeline failures will block merges.
- Violation handling: Code review comments will require remediation before approval.
- Exception process: Exceptions require explicit approval from a lead engineer or architect, documented with a clear rationale.