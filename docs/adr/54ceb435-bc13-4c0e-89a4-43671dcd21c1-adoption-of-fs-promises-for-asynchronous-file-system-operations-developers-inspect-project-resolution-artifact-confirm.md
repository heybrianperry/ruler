# Adoption of fs/promises for Asynchronous File System Operations: Developers Inspect Project Resolution Artifact Confirm

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The codebase coordinates agent interactions, skills processing, and subagent execution that require reading and persisting configuration and specification assets on disk.
- Six files across agent and core modules demonstrate consistent integration of fs/promises for file operations rather than blocking alternatives.
- Standardizing asynchronous file system access across modules prevents event loop contention during disk operations in processing pipelines.

## Problem Statement

Components within the agent and core processing layers require continuous access to filesystem resources, including agent instructions, skill definitions, and subagent configurations. Using disparate or synchronous file system access methods introduces thread-blocking latency and inconsistent error handling across modules, necessitating a standardized, promise-based file system interface.

## Decision

1. MUST: Developers MUST inspect the project resolution artifact to confirm the active runtime and dependency constraints before introducing or modifying file system integration code.

## Policy Block

- MUST Developers MUST inspect the project resolution artifact to confirm the active runtime and dependency constraints before introducing or modifying file system integration code.

In scope:
- All agent implementation modules requiring disk-based asset loading or configuration access.
- All core processor and utility modules performing file reading, parsing, or writing.

Out of scope:
- In-memory data manipulation routines that do not interface with physical file storage.
- Transient streaming network protocols that bypass local storage mechanisms.

## Rationale

- Static analysis shows uniform adoption of fs/promises across agent and processing boundaries, confirming a deliberate pattern of non-blocking file operations.
- Employing promise-based file access integrates cleanly with asynchronous control flow throughout agent execution lifecycles.
- Eliminating synchronous file operations prevents event loop starvation under high concurrency.

## Consequences

Positive:
- Non-blocking file operations preserve system responsiveness during concurrent processing tasks.
- Standardized promise interfaces align with modern asynchronous control flow constructs across all modules.
- Consistent file access patterns simplify debugging and error propagation across modular components.

Negative:
- Callers must handle asynchronous execution flow and promise rejections across all call sites.
- File read operations require explicit parsing and validation stages before downstream consumption.

## Alternatives

- Synchronous file system operations (rejected)
  Rejected because: Synchronous operations block the runtime event loop and degrade throughput during file-intensive execution cycles.
  When valid: Only permissible in one-time initialization scripts before the asynchronous event loop starts.
- Callback-based file system interfaces (rejected)
  Rejected because: Callback interfaces introduce nested control flow complexity and do not integrate cleanly with modern asynchronous syntax.
  When valid: When interfacing with legacy interfaces that strictly mandate callback signatures.

## Risks

- Unhandled promise rejections from failed file access can trigger unhandled runtime exceptions.
  Mitigation: Encapsulate file system promise calls in structured error-handling blocks with domain-specific recovery mechanisms.
  Owner: engineering team
- Concurrent read and write operations on identical file paths without coordination may cause data corruption.
  Mitigation: Coordinate shared file access through sequential processing stages or designated utility wrappers.
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
- Encapsulate fs/promises calls within dedicated utility modules or domain processors to centralize path resolution and encoding defaults.
- Pair file content retrieval with schema validation to verify file contents conform to expected formats before processing.

## Continuation Context


Verify commands:
- Discover the repository test runner script from the project manifest and execute the test suite covering file I/O operations.
- Run static analysis checks defined in the project build configuration to detect unapproved synchronous file system calls.

Accept when:
- All file operations in agent and core processing modules utilize fs/promises.
- Static analysis and automated test suites pass without detecting blocking file operations.

## Enforcement

- Verified by: Automated static analysis checks integrated into continuous integration pipelines.
- Verified by: Mandatory peer review for any pull request modifying or adding file system access patterns.
- Violation handling: Pull requests introducing synchronous or non-standard file operations will be blocked during automated verification.
- Violation handling: Violations identified in active branches must be refactored to use fs/promises prior to merging.
- Exception process: Exceptions require documented architectural justification and formal sign-off from principal engineers prior to implementation.