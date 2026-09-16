# Adoption of Node.js Core Modules and @iarna/toml for System Operations: Execution External Processes Performed Node Child

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires direct interaction with the underlying operating system for file system operations and process execution.
- Configuration files are formatted using TOML, necessitating a dedicated parsing library.
- Path manipulation is a common requirement across various core utilities.
- A local utility module FileSystemUtils is used to encapsulate common file system interactions.

## Problem Statement

The project needs a consistent and reliable approach for performing system-level operations, handling file paths, executing child processes, and parsing TOML configuration files within its core modules.

## Decision

1. MUST: Execution of external processes MUST be performed using the Node.js `child_process` module.

## Policy Block

- MUST Execution of external processes MUST be performed using the Node.js `child_process` module.

In scope:
- Core utility modules, engine components, and any code requiring direct system interaction, path manipulation, process execution, or TOML parsing.

Out of scope:
- Application-level logic not directly involved in system-level operations or configuration parsing.

## Rationale

- Node.js core modules (`fs`, `path`, `child_process`, `util`) provide fundamental, stable, and performant APIs for system-level interactions, which are essential for core functionalities.
- The `@iarna/toml` library is specifically adopted for parsing TOML, indicating a deliberate choice for handling this configuration format.
- Encapsulating file system operations within `FileSystemUtils` promotes reusability and maintainability, abstracting away direct `fs` calls.
- Standardizing on these modules ensures consistency and reduces the cognitive load for developers working on system-level aspects.

## Consequences

Positive:
- Consistent approach to file system, path, and process management.
- Reliable parsing of TOML configuration files.
- Leveraging well-tested and performant built-in Node.js capabilities.
- Improved maintainability through abstraction (e.g., `FileSystemUtils`).

Negative:
- Dependency on specific Node.js versions for core module behavior.
- Potential overhead if `child_process` is misused for frequent, short-lived tasks.

## Alternatives

- Implement custom file system utilities without relying on Node.js `fs` or `path`. (rejected)
  Rejected because: Reinventing core functionality is error-prone, less performant, and increases maintenance burden.
  When valid: For highly specialized, performance-critical scenarios where Node.js built-ins are insufficient, or for environments without Node.js.
- Use a different TOML parsing library. (rejected)
  Rejected because: `@iarna/toml` is already in use and meets current requirements; switching would incur migration costs without clear benefits.
  When valid: If `@iarna/toml` becomes unmaintained, has critical vulnerabilities, or lacks required features.

## Risks

- Misuse of `child_process` leading to security vulnerabilities (e.g., command injection) or performance issues.
  Mitigation: Implement strict input validation and sanitization for all `child_process` inputs; prefer safer alternatives like `execFile` over `exec` where possible; review `child_process` usage during code reviews.
  Owner: Engineering team
- Inconsistent or incorrect path handling across different operating systems.
  Mitigation: Rely exclusively on `path.join`, `path.resolve`, and other `path` module functions to ensure cross-platform compatibility; test on target operating systems.
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
- When abstracting core module functionality (e.g., `fs`), ensure the abstraction adds clear value (e.g., error handling, logging, specific business logic) rather than simply re-exporting.
- Prioritize asynchronous APIs for file system operations to prevent blocking the event loop.

## Continuation Context


Verify commands:
- Discover the project's static analysis configuration and execute it to identify direct imports of Node.js core modules and `@iarna/toml`.
- Locate the project's test suite and run all relevant tests to ensure system interactions and TOML parsing behave as expected.
- Examine the project's dependency manifest and lock file to confirm the presence and resolved version of `@iarna/toml`.

Accept when:
- Static analysis reports no unauthorized direct system interaction outside of established patterns.
- All tests related to file system operations, path handling, process execution, and TOML parsing pass successfully.
- The `@iarna/toml` library is present in the dependency graph at a resolved version.

## Enforcement

- Verified by: Automated CI checks, static analysis tools, and peer code reviews.
- Violation handling: Violations will block pull requests and require remediation before merging.
- Exception process: Exceptions require explicit approval from a lead architect or maintainer, documented with a clear rationale in a separate ADR or issue.