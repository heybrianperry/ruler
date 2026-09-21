# In-Memory Key-Value Caching Using Built-In Map Structures: Components Managing Transient Resolution State Utilize

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Configuration traversal and protocol server propagation routines require indexing resolved agents and server instances by directory or endpoint to prevent duplicate processing.
- Runtime executions utilize ephemeral in-memory map collections to cache intermediate agent mappings and endpoint records during lifecycle passes.
- The application architecture operates without a dedicated third-party caching datastore or persistent cache layer, relying directly on in-process data structures.

## Problem Statement

Runtime workflows that traverse directory hierarchies and process server definitions perform repetitive lookups for identical directories and network endpoints. Executing redundant resolution cycles incurs unnecessary computational overhead and potential configuration divergence across sequential evaluation phases within a single execution cycle.

## Decision

1. MUST: Components managing transient resolution state MUST utilize localized in-memory key-value map structures with set and get operations to isolate runtime cache entries within the active process execution scope.

## Policy Block

- MUST Components managing transient resolution state MUST utilize localized in-memory key-value map structures with set and get operations to isolate runtime cache entries within the active process execution scope.

In scope:
- In-memory caching and transient indexing of configuration entries, agent definitions, and server endpoints during single-run execution workflows.
- Localized key-value lookups within runtime evaluation modules.

Out of scope:
- Persistent data storage across distinct process invocations.
- Distributed or multi-process state synchronization requiring external primary datastores.

## Rationale

- Direct usage of standard in-memory key-value maps provides zero-dependency caching for short-lived execution passes without operational overhead.
- Keying intermediate records by configuration path or endpoint identifier eliminates redundant resolution cycles within directory traversal passes.
- Keeping cache boundaries localized to execution passes prevents cross-test pollution and avoids stale configuration state across independent runs.

## Consequences

Positive:
- Eliminates redundant filesystem and configuration evaluations within single execution runs.
- Avoids external infrastructure dependencies and network latency for transient lookups.
- Simplifies state management through direct in-process map lookups.

Negative:
- Cache contents are lost immediately upon process termination and cannot be shared across concurrent worker processes.
- Absence of built-in expiration or eviction mechanisms creates memory growth risk if applied to long-running unbounded processes.

## Alternatives

- Adopting an external distributed cache or primary datastore service (rejected)
  Rejected because: Introduces unnecessary operational overhead, external service dependencies, and network serialization costs for CLI commands and transient evaluation runs.
  When valid: Valid when multiple concurrent worker processes must synchronize shared state across separate hosts or processes.
- Re-evaluating configurations and endpoints on every query without caching (rejected)
  Rejected because: Results in repeated disk access and redundant parsing overhead during recursive directory evaluations.
  When valid: Valid in stateless utility functions with negligible computational cost.

## Risks

- Unbounded growth of in-memory maps if retained in persistent long-running execution contexts
  Mitigation: Scope cache instances strictly to the lifetime of single execution requests or commands.
  Owner: Engineering Team
- Stale data returned if underlying filesystem configurations change during an extended process lifetime
  Mitigation: Clear or recreate cache collections whenever a mutation or file system modification is triggered.
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
- Instantiate cache map instances at the top of the evaluation workflow and pass them down or encapsulate them within the resolution coordinator to ensure proper garbage collection after execution.
- Use normalized absolute string paths or canonical URLs as lookup keys to avoid cache misses caused by equivalent but differently formatted keys.

## Continuation Context


Verify commands:
- Discover and run the project's test suite to verify that in-memory cache operations preserve resolution correctness.
- Execute the repository's static type checker and linter scripts to validate cache map key-value typings.

Accept when:
- All unit and integration tests passing without memory leaks or state bleeding across test cases.
- Static analysis and type checking complete with zero errors regarding cache datastore interactions.

## Enforcement

- Verified by: Automated continuous integration test execution checking for test isolation and deterministic behavior.
- Verified by: Peer code review verifying that cache lifecycles remain bounded to appropriate execution contexts.
- Violation handling: Pull requests introducing global mutable cache state without lifecycle bounds will be rejected during review.
- Exception process: Exceptions requiring persistent or cross-process datastores require an architectural review proposal submitted to the engineering team.