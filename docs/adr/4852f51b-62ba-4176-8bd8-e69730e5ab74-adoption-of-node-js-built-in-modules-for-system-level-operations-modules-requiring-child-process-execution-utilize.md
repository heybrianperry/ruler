# Adoption of Node.js Built-in Modules for System-Level Operations: Modules Requiring Child Process Execution Utilize

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- Project requires direct interaction with the underlying operating system for tasks.
- File system operations are necessary for managing project-specific files and configurations.
- Execution of external commands or scripts is integrated into development or testing workflows.
- Utility functions are encapsulated in local modules for reusability.

## Problem Statement

The project needs a standardized approach for performing system-level operations, including file system access, path manipulation, and child process execution, ensuring consistency and maintainability across relevant modules.

## Decision

1. MUST: Modules requiring child process execution MUST utilize the Node.js `child_process` module.

## Policy Block

- MUST Modules requiring child process execution MUST utilize the Node.js `child_process` module.

In scope:
- Modules responsible for file system operations, process management, and core utilities.

Out of scope:
- Frontend code, business logic not directly interacting with the OS.

## Rationale

- Leverages native Node.js capabilities for efficient system-level interactions.
- Promotes consistency by centralizing system operation logic through standard modules.
- Reduces external dependencies by utilizing built-in functionality.

## Consequences

Positive:
- Consistent approach to system interactions.
- Reduced overhead from third-party libraries for basic system tasks.
- Improved performance for file and process operations.

Negative:
- Direct exposure to Node.js API specifics requires developer familiarity.
- Potential for platform-specific issues if not carefully abstracted.

## Alternatives

- Use third-party libraries for file system and process management. (rejected)
  Rejected because: Introduces unnecessary external dependencies and overhead for functionality already provided by Node.js built-ins.
  When valid: For highly complex or cross-platform system interactions that built-ins cannot easily handle.

## Risks

- Inconsistent error handling across different system operations.
  Mitigation: Establish project-wide error handling patterns for system calls.
  Owner: engineering team
- Security vulnerabilities from improper use of `child_process`.
  Mitigation: Strictly validate all inputs to child processes and use safe execution methods.
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
- Encapsulate complex system interactions within dedicated utility functions to promote reusability and testability.
- Ensure proper resource management (e.g., closing file handles) for `fs` operations.

## Continuation Context


Verify commands:
- Inspect relevant module import statements for `fs`, `path`, `child_process`, and `util`.
- Review code for direct usage of `child_process` APIs.
- Examine test setup files for system-level module imports.

Accept when:
- All file system operations use the `fs` module.
- All path manipulations use the `path` module.
- All child process executions use the `child_process` module.

## Enforcement

- Verified by: Automated static analysis in CI/CD pipelines and code reviews.
- Violation handling: Flagged during code review, CI/CD build failure.
- Exception process: Requires explicit approval from architectural review board with documented justification.