# In-Memory Map Cache for Transient Entity Lookups: Modules Utilizing Memory Key Value Lookups

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Transient key-value lookups are used across modules to memoize filesystem checks and index configuration entities during runtime operations.
- The codebase relies on standard built-in collection instances rather than integrating an external caching service or dedicated datastore client.
- Observed call patterns directly invoke collection set and get methods for ephemeral state retention within individual module lifecycles.

## Problem Statement

Uncoordinated, ad hoc usage of in-memory collection primitives across disparate modules risks unbounded memory retention and inconsistent cache eviction, requiring explicit architectural governance over whether in-memory structures or a formal caching datastore should manage transient state.

## Decision

1. MUST: Modules utilizing in-memory key-value lookups MUST implement explicit size boundaries or lifecycle cleanup to prevent unbounded memory growth.

## Policy Block

- MUST Modules utilizing in-memory key-value lookups MUST implement explicit size boundaries or lifecycle cleanup to prevent unbounded memory growth.

In scope:
- In-memory caching of entity configurations and file path resolution states within local module execution lifecycles.
- Transient state indexing during single-process operations.

Out of scope:
- Persistent data storage requiring durability across process restarts.
- Distributed state sharing across multi-process or networked application instances.

Exceptions:
- EXC-28-001: A module requires short-lived memoization within a single synchronous call stack where garbage collection reclaims state immediately upon return.

## Rationale

- Direct usage of built-in collection primitives introduces zero runtime dependency overhead for transient lookups.
- Restricting built-in collection caches to ephemeral lifecycles prevents memory leaks caused by unbounded growth in long-running processes.
- Distinguishing localized memoization from primary datastore requirements ensures that shared persistence needs are handled by dedicated datastore layers rather than accidental state accumulation.

## Consequences

Positive:
- Avoids external infrastructure and third-party dependency overhead for simple, localized entity lookups.
- Ensures fast, in-process key-value retrieval latency without network serialization costs.
- Clarifies architectural boundaries between transient in-memory memoization and durable primary datastore persistence.

Negative:
- State is not shared across processes or preserved across application restarts.
- Lacks built-in cache invalidation, expiration policies, and automatic memory eviction mechanisms.

## Alternatives

- Adopt an external distributed key-value datastore for all cached entities (rejected)
  Rejected because: Introduces significant infrastructure complexity, network latency, and operational overhead for ephemeral, process-local lookups.
  When valid: When cached entities must be shared across multiple distributed worker nodes or survive process restarts.
- Adopt a specialized in-process cache library with automated least-recently-used eviction (deferred)
  Rejected because: Current memory footprint and lookup volume do not yet justify an external third-party caching dependency.
  When valid: When in-memory cache sizes grow unbounded or require time-to-live expiration policies.

## Risks

- Unbounded in-memory collections can lead to memory exhaustion in long-running service processes.
  Mitigation: Enforce lifecycle scoping, manual clearing, or weak references on long-lived collection instances.
  Owner: Engineering Team
- Stale cache entries may persist when underlying configuration or filesystem entities change.
  Mitigation: Invalidate or reinitialize cache mappings whenever source configuration files or paths are mutated.
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
- Scope collection instances to the shortest viable lifespan, preferentially instantiating caches inside operation handlers rather than at module scope.
- Verify that asynchronous operations populating cache entries handle rejection cleanly to prevent storing incomplete or corrupt state.

## Continuation Context


Verify commands:
- find . -type f -exec grep -E "\.(set|get)\(" {} +
- find . -maxdepth 2 -type f -exec grep -E '"test":|"scripts":' {} +

Accept when:
- All in-memory cache structures have defined lifecycle clearing or are scoped to transient operation execution.
- Project verification and test suites pass without memory leakage or stale cache regression errors.

## Enforcement

- Verified by: Peer code review during pull request evaluation.
- Verified by: Automated static analysis and unit testing in continuous integration pipelines.
- Violation handling: Flagged during code review for remediation before merging.
- Violation handling: Refactoring required to bound collection lifecycles or introduce proper eviction semantics.
- Exception process: Submit an architectural review request detailing why an unbounded in-memory collection is required.
- Exception process: Obtain documented approval from the architecture review group with an assigned review date.