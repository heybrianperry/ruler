# child_process Module Adoption for Subprocess Management: Subprocess Invocations That Consume Dynamic Input

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires interactions with external system utilities and host operating system binaries for core functionality.
- Direct invocation of external binaries is required when native in-process libraries do not provide equivalent operational capabilities.
- Test suites require predictable execution environments that isolate unit and integration tests from external host system dependencies.

## Problem Statement

Executing external operating system binaries and external utilities requires a structured process invocation mechanism. Without a standardized subprocess execution pattern, components risk inconsistent error handling, vulnerability to command injection, resource leaks from unmanaged child processes, and brittle test suites dependent on host system configuration.

## Decision

1. MUST: Subprocess invocations that consume dynamic input MUST sanitize all arguments and avoid shell execution interpreters to prevent command injection vulnerabilities.

## Policy Block

- MUST Subprocess invocations that consume dynamic input MUST sanitize all arguments and avoid shell execution interpreters to prevent command injection vulnerabilities.

In scope:
- Components requiring invocation of host operating system binaries or external system utilities.
- Test setup files configuring mocks and stubs for subprocess execution.

Out of scope:
- Operations that can be fully satisfied using pure in-process libraries or standard data manipulation routines.
- Client-side or browser-targeted modules where process spawning is not supported by the runtime.

Exceptions:
- EXC-20-001: A module requires direct native bindings or operating system API hooks that cannot be fulfilled via subprocess execution.

## Rationale

- Static analysis identifies the child_process module imported in both core utilities and test setup files, confirming its role as the standard subprocess mechanism.
- Standardizing on the child_process module provides centralized control over process lifecycles, stream handling, and security boundaries across runtime and testing environments.
- Isolating subprocess execution allows unit test suites to substitute predictable stubs during verification while maintaining consistent production behavior.

## Consequences

Positive:
- Provides a standard, runtime-native interface for spawning and managing external system processes without third-party wrapper dependencies.
- Enables isolation of unit tests through structured mocking of child process execution streams and return codes.
- Facilitates consistent security controls, argument escaping, and timeout management across process boundaries.

Negative:
- Introduces runtime coupling to host operating system environments and externally installed executables.
- Requires defensive error handling and buffer management to prevent blocked streams or zombie child processes.
- Increases testing overhead due to the necessity of mocking process communication interfaces in test harnesses.

## Alternatives

- Pure In-Process Reimplementation (rejected)
  Rejected because: Reimplementing complex operating system utility logic in pure application code increases maintenance burden and code complexity significantly.
  When valid: Valid when lightweight in-process routines exist that completely satisfy functional requirements without binary execution.
- Third-Party Subprocess Wrapper Libraries (rejected)
  Rejected because: Adding external wrapper dependencies increases dependency surface area without providing critical functionality beyond the standard child_process module.
  When valid: Valid when advanced process orchestration, cross-platform terminal emulation, or complex pipeline piping is required across services.

## Risks

- Subprocess execution hangs indefinitely due to unhandled standard I/O buffer saturation or unmanaged process lifecycles.
  Mitigation: Enforce strict execution timeouts and attach error and close event listeners to all spawned process instances.
  Owner: engineering team
- Command injection vulnerabilities resulting from unescaped dynamic arguments passed to command shells.
  Mitigation: Avoid shell invocation mode and pass arguments strictly as disjoint array elements directly to the binary.
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
- When executing subprocesses, prefer argument array passing over shell string evaluation to protect against process execution exploits.
- Ensure test harness setup configurations intercept child process module exports to provide deterministic fixture responses during test runs.

## Continuation Context


Verify commands:
- find . -type f \( -name "*.ts" -o -name "*.js" \) -not -path "*/.*" -exec grep -l "child_process" {} +
- SCRIPT=$(cat <(find . -maxdepth 2 -name "*manifest*" -o -name "*json*") 2>/dev/null | grep -o '"test": *"[^"]*"' | head -n 1 | cut -d'"' -f4) && [ -n "$SCRIPT" ] && sh -c "$SCRIPT"

Accept when:
- All references to subprocess invocation resolve exclusively through the child_process module.
- Test suites pass successfully with subprocess mocks in place and without unexpected child process termination.

## Enforcement

- Verified by: Automated static analysis checks and test suite runs in continuous integration pipelines.
- Verified by: Mandatory architectural peer review of any pull request introducing or modifying subprocess execution calls.
- Violation handling: Code review rejection and failure of automated continuous integration checks for unsanitized subprocess calls.
- Violation handling: Requirement to refactor non-standard process execution mechanisms to adhere to child_process conventions.
- Exception process: Submit an architectural exemption request detailing technical constraints, security mitigations, and lifecycle controls to the architecture team.