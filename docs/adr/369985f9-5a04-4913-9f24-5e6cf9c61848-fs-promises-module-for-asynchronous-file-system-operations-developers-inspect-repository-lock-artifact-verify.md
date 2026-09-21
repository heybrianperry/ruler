# fs/promises Module for Asynchronous File System Operations: Developers Inspect Repository Lock Artifact Verify

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Multiple subsystems within the codebase perform persistent disk operations, including configuration management, protocol synchronization, and agent workspace manipulation.
- Synchronous disk I/O blocks the single-threaded event loop, degrading responsiveness and throughput during concurrent task execution.
- Adopting the promise-based file system module establishes a standard asynchronous programming model across command-line interfaces, protocol propagators, and agent workflows.

## Problem Statement

File I/O operations across application layers risk blocking the runtime event loop if synchronous APIs are used, leading to degraded performance and potential deadlocks during concurrent operations. The system requires a consistent, non-blocking asynchronous standard for all disk interactions to maintain responsiveness and ensure reliable error handling.

## Decision

1. MUST: Developers MUST inspect the repository lock artifact to verify the exact resolved version of any versioned runtime dependencies or typing packages before implementing or modifying file system interfaces.

## Policy Block

- MUST Developers MUST inspect the repository lock artifact to verify the exact resolved version of any versioned runtime dependencies or typing packages before implementing or modifying file system interfaces.

In scope:
- All application modules that read from, write to, or traverse disk file systems.
- Command-line handlers, model context protocol synchronizers, and autonomous agent persistence layers.

Out of scope:
- Purely in-memory caching and transient state management.
- External network I/O or remote streaming protocols not involving the local file system.

## Rationale

- Non-blocking I/O provided by fs/promises ensures the event loop remains responsive during heavy file reads and writes across concurrent processes.
- Promise-based APIs seamlessly integrate with modern asynchronous control flows and structured try-catch error handling.
- Standardizing on fs/promises prevents accidental introduction of thread-blocking synchronous file calls.

## Consequences

Positive:
- Eliminates main-thread blocking during configuration loading, file parsing, and state persistence.
- Provides unified asynchronous error propagation through standard promise rejections.
- Enables concurrent asynchronous file reads and writes without thread pool starvation.

Negative:
- Requires asynchronous function signatures across call chains that interact with disk files.
- Introduces the risk of unhandled promise rejections if call sites fail to catch asynchronous errors.

## Alternatives

- Synchronous file system API calls (rejected)
  Rejected because: Synchronous file operations block the runtime event loop, degrading performance and responsiveness across concurrent operations.
  When valid: Only permissible in ephemeral bootstrap scripts prior to application runtime initialization where event-loop blocking is inconsequential.
- Legacy callback-based file system APIs (rejected)
  Rejected because: Callback-based APIs result in nested control flows and fragmented error handling compared to promise-based async/await syntax.
  When valid: When integrating with legacy external libraries that strictly accept callback signatures without promise support.

## Risks

- Unhandled promise rejections from disk read or write failures causing unexpected process termination.
  Mitigation: Enforce structured try-catch blocks and automated rejection trapping across all asynchronous file operations.
  Owner: Engineering Team
- Race conditions during concurrent writes to the same configuration or state file.
  Mitigation: Implement sequential write coordination or atomic write patterns through temporary file swap strategies.
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
- Wrap asynchronous file system interactions in structured try-catch blocks or chain rejection handlers to prevent unhandled promise rejections during disk read or write failures.
- Utilize companion path resolution modules to canonicalize file system targets before executing promise-based file operations.

## Continuation Context


Verify commands:
- Discover the repository test runner script from the project manifest and execute the test suite validating file operations and asynchronous handler execution.
- Discover the static analysis and linting script from the project manifest and run it to verify that no synchronous file system APIs are invoked across the codebase.

Accept when:
- All asynchronous file system operations execute without unhandled promise rejections.
- Automated test suites pass with zero regressions across file-dependent modules.
- Static analysis confirms zero invocations of blocking synchronous file system functions.

## Enforcement

- Verified by: Automated continuous integration pipelines executing test suites and static analysis.
- Verified by: Peer code reviews validating adherence to promise-based file system conventions and error handling.
- Violation handling: Pull requests containing synchronous file system invocations or unhandled file promise rejections must be blocked from merging.
- Violation handling: Flagged violations must be refactored to use asynchronous fs/promises methods with explicit error handling.
- Exception process: Exceptions for synchronous operations in early initialization bootstrap require written architectural review approval and justification.
- Exception process: Any approved exception must be documented alongside mitigation for event loop blocking.