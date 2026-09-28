# Adoption of child_process Module for Subprocess Execution: Utility Functions Wrapping Child Process Invocations

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires execution of external operating system processes to perform environment initialization and repository-level utility operations.
- Interacting with platform binaries and process-level utilities necessitates low-level operating system process management.
- The project relies on the core runtime module child_process across core utility modules and test harness initialization routines instead of introducing external process execution wrappers.

## Problem Statement

Executing external system processes and shell utilities without standardized conventions leads to inconsistent process lifecycle handling, unhandled stream buffering errors, and unpredictable cross-environment execution behavior. The application requires a consistent, dependency-free approach for managing subprocess invocation, stream handling, and error propagation across utility libraries and test infrastructure.

## Decision

1. SHOULD: Utility functions wrapping child_process invocations SHOULD encapsulate subprocess lifecycle management within structured abstractions to provide centralized error handling.

## Policy Block

- SHOULD Utility functions wrapping child_process invocations SHOULD encapsulate subprocess lifecycle management within structured abstractions to provide centralized error handling.

In scope:
- Components requiring interaction with local operating system binaries and host command execution.
- Test harness and environment initialization routines needing sub-process orchestration.

Out of scope:
- Pure computation and in-memory data processing modules where operating system processes are not required.
- Cross-network remote procedure calls and service communications governed by network protocols.

## Rationale

- Static code analysis reveals imports and invocations of child_process across both core utility modules and test initialization layers, establishing it as the standard mechanism for sub-process execution.
- Utilizing the built-in runtime module minimizes third-party dependency bloat and eliminates supply chain risk associated with external process-wrapping libraries.
- Standardizing on child_process ensures consistent process supervision, input sanitization, and stream handling across the repository.

## Consequences

Positive:
- Zero external dependencies are introduced for process spawning and command orchestration.
- Direct access to low-level operating system stream controls and exit code monitoring.
- Unified process invocation conventions across both runtime utilities and testing setups.

Negative:
- Requires manual management of asynchronous stream buffers and error handling compared to higher-level opinionated abstractions.
- Increases the surface area for platform-specific command execution differences across host operating systems.

## Alternatives

- Adopting third-party process execution helper libraries (rejected)
  Rejected because: Introducing external libraries increases dependency bloat and supply-chain exposure when standard runtime modules provide sufficient capabilities.
  When valid: When complex cross-platform stream piping and terminal emulation features are required beyond standard execution.
- Relying exclusively on native host shell scripts outside the application runtime (rejected)
  Rejected because: Decouples process orchestration from application lifecycle management and complicates programmatic output parsing.
  When valid: For initial environment bootstrapping prior to runtime installation.

## Risks

- Improper sanitization of command arguments passed to shell execution methods can expose command injection vulnerabilities.
  Mitigation: Enforce argument array parameterization via spawn or execFile rather than shell string interpolation, verified through code review.
  Owner: engineering team
- Unbounded buffer consumption on standard output or standard error streams can cause process deadlocks.
  Mitigation: Use streaming interfaces with explicit buffer limits or consume streams incrementally.
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
- Encapsulate recurring subprocess invocations within dedicated utility wrapper functions to centralize stream parsing and error propagation.
- Ensure all child processes are gracefully terminated upon parent process teardown to prevent orphaned zombie processes.

## Continuation Context


Verify commands:
- Discover and run the repository test suite verifying that utility functions and test setup scripts execute subprocess calls without error.
- Discover and execute the repository static analysis and security scanning tasks to verify secure invocation patterns of process execution methods.

Accept when:
- All tests verifying subprocess operations pass with zero unhandled stream errors or non-zero exit code failures.
- Static analysis checks confirm no unsanitized command string concatenations are present in subprocess execution calls.

## Enforcement

- Verified by: Automated continuous integration test suites and static analysis linting rules.
- Verified by: Peer code review checking argument parameterization and stream handling on all new subprocess invocations.
- Violation handling: Pull requests containing unhandled process errors or direct shell string concatenation will be blocked until remediated.
- Exception process: Exceptions requiring third-party process management libraries must be submitted via an architecture review proposal detailing technical justification.