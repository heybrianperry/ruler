# Subprocess Execution via child_process: Subprocess Invocation Methods Not Concatenate Unvalidated

Status: proposed
Date: 2025-02-18
Deciders: Detection Pipeline (automated)

## Context

- Core system utilities require interaction with host platform binaries and external tooling to inspect runtime state and execute commands.
- Test infrastructure components require environment preparation and subprocess orchestration during test suite initialization.
- Static intermediate representation analysis reveals repeated usage of the child_process module across both core utilities and test environment setup.

## Problem Statement

Executing external system commands and inspecting environment facilities directly without a standardized architectural pattern risks inconsistent process lifecycle management, unhandled stream failures, potential command injection vulnerabilities, and tight coupling to host operating system environments.

## Decision

1. MUST_NOT: Subprocess invocation methods MUST NOT concatenate unvalidated or untrusted external input strings into shell command execution strings.

## Policy Block

- MUST_NOT Subprocess invocation methods MUST NOT concatenate unvalidated or untrusted external input strings into shell command execution strings.

In scope:
- Core utility modules performing system-level inspection and external process execution.
- Test harness initialization scripts orchestrating environment setup and external processes.

Out of scope:
- In-memory data transformations and domain business logic not requiring host operating system interaction.
- Standard library filesystem read and write operations that do not require external process invocation.

Exceptions:
- EXC-20-001: Synchronous subprocess execution is strictly required during initial environment bootstrap before the asynchronous loop begins.

## Rationale

- Direct usage of the child_process module provides standard access to host execution capabilities without introducing external third-party dependencies.
- Centralizing external command execution under explicit lifecycle and stream management prevents runaway background processes and unhandled failure states.
- Enforcing separation between execution interfaces and calling modules enables deterministic testing and environmental portability.

## Consequences

Positive:
- Consistent and controlled subprocess lifecycle management across runtime and test environments.
- Elimination of external wrapper dependencies by standardizing on the core child_process module.
- Improved security posture against command injection through mandatory argument separation.

Negative:
- Subprocess creation incurs process initialization and inter-process communication overhead.
- Execution behavior remains inherently dependent on the underlying host operating system and path resolution.

## Alternatives

- Adopting an external third-party process execution wrapper library (rejected)
  Rejected because: Introduces extraneous external dependencies when core platform module capabilities sufficiently satisfy current architectural requirements.
  When valid: When advanced cross-platform stream multiplexing or complex process pooling is required.
- In-process reimplementation of external binary tool logic (rejected)
  Rejected because: Reimplementing full binary functionality in-process requires excessive maintenance overhead and risks behavioral discrepancies.
  When valid: When the required functionality is minimal and performance constraints make process spawning unacceptable.

## Risks

- Subprocesses may hang indefinitely if the spawned process blocks on input or fails to terminate.
  Mitigation: Enforce explicit timeout configurations and signal-based cancellation on all process invocations.
  Owner: engineering team
- Command injection vulnerabilities arising from improper argument formatting.
  Mitigation: Prohibit raw shell string concatenation and mandate argument array passing for subprocess execution.
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
- Verify that all subprocess invocation calls receive command arguments as distinct array elements rather than interpolated shell strings.
- Ensure standard error and standard output streams are drained or piped properly to avoid buffer exhaustion deadlocks.

## Continuation Context


Verify commands:
- Discover the project test execution script from the repository manifest and run test suites covering subprocess execution.
- Inspect the project configuration to identify and run the static analysis and linting scripts against module boundaries.

Accept when:
- All test suites verifying subprocess invocation and error handling pass with exit code zero.
- Static verification confirms all child_process invocations provide explicit error handling and timeout boundaries.

## Enforcement

- Verified by: Automated continuous integration pipeline running test suites and static analysis checks.
- Verified by: Peer code review verifying adherence to process execution rules and input sanitization.
- Violation handling: Build failures in automated continuous integration pipelines on unhandled process executions.
- Violation handling: Code review rejection for raw string concatenation or missing timeout parameters in process execution calls.
- Exception process: Submit an architectural deviation request detailing the technical constraint and security review before merging.