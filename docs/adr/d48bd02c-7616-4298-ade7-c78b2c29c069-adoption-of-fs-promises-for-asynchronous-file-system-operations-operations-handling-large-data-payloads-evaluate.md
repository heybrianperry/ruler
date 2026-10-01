# Adoption of fs/promises for Asynchronous File System Operations: Operations Handling Large Data Payloads Evaluate

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Multiple subsystems across command-line handlers, agent orchestration, protocol propagation, and core utilities interact directly with host storage.
- Synchronous disk input and output operations risk blocking execution on the primary thread, creating latency bottlenecks across concurrent tasks.
- Internal components have converged on asynchronous promise interfaces to maintain unified control flow semantics with async-await language constructs.
- A standardized file persistence approach prevents fragmentation between conflicting synchronous and asynchronous storage patterns.

## Problem Statement

Uncoordinated file system interaction across command-line interfaces, agents, and protocol propagators risks thread blocking and uneven error handling. The architecture requires an explicit, non-blocking standard for all local file operations.

## Decision

1. SHOULD: Operations handling large data payloads SHOULD evaluate stream-oriented interfaces rather than loading complete file contents into memory with single promise invocations.

## Policy Block

- SHOULD Operations handling large data payloads SHOULD evaluate stream-oriented interfaces rather than loading complete file contents into memory with single promise invocations.

In scope:
- All internal modules, command-line handlers, agent orchestrators, and protocol propagation components performing file system access.
- Any operation reading, writing, updating, or querying the host file system.

Out of scope:
- In-memory state manipulation that does not persist to disk.
- Network-only protocol communication without local file system backing.

Exceptions:
- EXC-20-001: A bootstrap initialization sequence strictly requires synchronous configuration loading before the asynchronous runtime loop initializes.

## Rationale

- Adopting fs/promises enforces asynchronous, non-blocking I/O across storage-interacting subsystems, preserving event loop responsiveness during disk operations.
- Promise-based APIs integrate directly with language-native async and await constructs, simplifying error handling hierarchies and sequential file workflows.
- Standardizing on fs/promises across CLI handlers, agent components, and protocol propagation logic eliminates fragmented utility patterns and promotes maintainable storage abstractions.

## Consequences

Positive:
- Prevents event loop starvation by guaranteeing non-blocking I/O during heavy file manipulation.
- Unifies error propagation and asynchronous control flow across disparate modules.
- Aligns file operations with modern promise-oriented language interfaces.

Negative:
- Requires asynchronous function signatures across call stacks that initiate file operations.
- Requires deliberate resource management and stream handling when processing large file payloads.

## Alternatives

- Synchronous file system execution (rejected)
  Rejected because: Synchronous execution blocks the single-threaded event loop, degrading throughput and responsiveness across concurrent operations.
  When valid: Valid only in immediate startup scripts prior to asynchronous event loop execution.
- Callback-based file system APIs (rejected)
  Rejected because: Callback interfaces introduce nested control flow complexity and complicate unified error handling relative to promise-based async-await patterns.
  When valid: Valid in legacy environments lacking native promise support.

## Risks

- Uncaught asynchronous rejection during file access leading to unhandled promise rejections.
  Mitigation: Mandate structured try-catch or promise rejection handlers across all file access boundaries.
  Owner: engineering team
- Memory exhaustion when reading unbounded file contents directly into memory via single promise calls.
  Mitigation: Enforce streaming or chunked processing strategies for arbitrarily large data files.
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
- Ensure all asynchronous file system methods are paired with appropriate error handling to catch missing files, permission errors, and invalid path inputs.
- Coordinate file path generation with path resolution utilities to avoid operating-system-specific path separator issues.

## Continuation Context


Verify commands:
- Discover the project test runner from repository configuration and execute the automated test suite.
- Discover the static analysis tool from repository configuration and run linting checks to identify disallowed synchronous file system calls.

Accept when:
- All automated test suites pass without regression in asynchronous execution.
- Static analysis validates that no synchronous file system calls exist in asynchronous execution paths.

## Enforcement

- Verified by: Automated static analysis checks in the continuous integration pipeline.
- Verified by: Peer code review for pull requests modifying storage or file manipulation layers.
- Violation handling: Continuous integration failure and rejection of pull requests containing unauthorized synchronous file system calls.
- Exception process: Submit an architectural review request documenting the technical necessity for an alternative file access pattern.