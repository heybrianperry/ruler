# In-Memory Map Key-Value Caching: Transient Memory Cache Structures Not Used

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Runtime workflows require fast retrieval of configuration entries and resolved agent definitions during execution cycles.
- The codebase utilizes standard in-memory map data structures to index and store entities by key within local operational modules.
- The implementation operates without introducing external distributed datastores or third-party cache dependencies.

## Problem Statement

Repeated resolution and lookup of configuration entities across multi-step execution flows introduce redundant computation if entities are not retained in memory during the operational lifecycle.

## Decision

1. MUST_NOT: Transient in-memory cache structures MUST NOT be used for data that requires persistence across separate application process lifecycles.

## Policy Block

- MUST_NOT Transient in-memory cache structures MUST NOT be used for data that requires persistence across separate application process lifecycles.

In scope:
- In-memory caching and transient lookup tables within local module execution workflows.
- Entity indexing for configuration entries and resolved task components during execution.

Out of scope:
- Long-term data persistence across distinct application lifecycles or distributed nodes.
- External database integrations and durable file-system storage.

## Rationale

- Using language built-in map collections provides zero-dependency, constant-time entity lookup during localized task execution.
- Local in-memory key-value structures prevent recurring parsing and resolution costs without incurring external cache management complexity.
- Confining storage to execution scope prevents unintended state leakage across detached operational boundaries.

## Consequences

Positive:
- Eliminates repetitive entity resolution and configuration parsing within execution routines.
- Avoids external infrastructure dependencies and operational overhead for transient data storage.
- Ensures constant-time access complexity for keyed lookups.

Negative:
- Data does not persist across separate process invocations or application restarts.
- Memory footprint scales with the number of cached entities unless explicit lifecycle cleanup is enforced.
- Lacks built-in cache expiration policies, time-to-live management, and size boundaries.

## Alternatives

- Adopting an external dedicated caching daemon or database (rejected)
  Rejected because: Introduces unnecessary operational dependencies and network latency for data that only needs to exist during a single execution run.
  When valid: Valid when cached state must be shared across distributed processes or survive process termination.
- Re-computing and re-parsing entity configurations on every access (rejected)
  Rejected because: Creates redundant computation overhead and file reading during iterative task workflows.
  When valid: Valid when entity instances are rarely accessed and memory constraints preclude caching.

## Risks

- Unbounded growth of in-memory map entries leading to elevated memory utilization in prolonged execution contexts.
  Mitigation: Scope map lifecycles to individual execution routines or implement explicit cache clearance upon workflow completion.
  Owner: Engineering Team
- Stale data retention if underlying configuration sources change during an ongoing execution session.
  Mitigation: Re-initialize in-memory lookup maps whenever underlying configuration sources are reloaded or modified.
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
- Instantiate lookup maps within function or module scopes where entity resolution is contained, avoiding export of mutable map references.
- Ensure key derivation functions generate collision-free identifiers for cached items.

## Continuation Context


Verify commands:
- Discover and run the project static analysis and linting scripts to verify compliance with local state encapsulation rules.
- Discover and execute the project automated test suite to confirm that cache population and retrieval logic passes all unit and integration tests.

Accept when:
- All automated test suites pass without regressions in configuration resolution or execution workflows.
- Static analysis checks complete with zero errors regarding unmanaged mutable global state.

## Enforcement

- Verified by: Automated test validation within the continuous integration pipeline.
- Verified by: Peer code review during pull request evaluations.
- Violation handling: Flagged pull requests introducing unmanaged global caches or external datastore dependencies without architectural approval must be revised before merge.
- Exception process: Exceptions requiring persistent or distributed caching must be documented and submitted via a formal architectural review.