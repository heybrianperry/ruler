# In-Memory Map-Based Cache Layer Pattern: Cache Population Use Set Method Identifier

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase manages server connection state (SSE, SHTTP, STDIO servers) and configuration hierarchies (agent selections by ruler directory) where the same identifiers may be queried multiple times within a single operation lifecycle
- Repeated parsing of configuration files, instantiation of server connections, or resolution of agent selections for the same identifier would introduce unnecessary I/O overhead and potential inconsistency
- The project requires a lightweight caching mechanism that operates within the memory space of a single process without introducing external dependencies or persistence requirements
- Three distinct modules (MCP server propagation, revert operations, core library logic) independently implement the same structural pattern using native Map data structures for lookup table caching

## Problem Statement

The system needs to avoid redundant computation, I/O operations, and object instantiation when the same configuration identifiers (URLs, directory paths, server names) are accessed multiple times during request processing or operation execution. Without a consistent caching strategy, each module would implement ad-hoc memoization approaches, leading to inconsistent cache semantics, duplicated logic, and potential bugs from cache invalidation mismatches.

## Decision

1. MUST: Cache population MUST use the set method with the identifier as key and the computed result (server entry, agent array, configuration object) as value

## Policy Block

- MUST Cache population MUST use the set method with the identifier as key and the computed result (server entry, agent array, configuration object) as value

In scope:
- Server connection management where the same URL or server name may be referenced multiple times
- Configuration hierarchy resolution where the same directory path may be queried repeatedly
- Any subsystem that repeatedly queries the same identifiers within a request or operation lifecycle
- Scenarios where cached values are complex objects (server entries, agent arrays, configuration objects) that are expensive to compute or instantiate

Out of scope:
- Cross-process caching requirements where state must be shared between multiple application instances
- Persistent caching where cached values must survive process restarts
- Distributed caching scenarios requiring cache coherence across multiple nodes
- Cache invalidation strategies requiring time-based expiration or event-driven eviction beyond operation scope

## Rationale

- The pattern is observed consistently across three distinct modules (MCP server propagation, revert operations, core library logic), indicating an established architectural convention rather than isolated implementation choices
- Using native Map data structures avoids introducing external caching library dependencies while providing O(1) lookup performance and straightforward semantics for key-value storage
- The evidence shows caching applied to configuration state and connection management, where repeated resolution of the same identifier would cause redundant I/O (file parsing) or instantiation overhead
- Map-based caching provides sufficient functionality for operation-scoped memoization without the complexity of TTL management, eviction policies, or serialization required by more sophisticated caching solutions

## Consequences

Positive:
- Eliminates redundant parsing, I/O operations, and object instantiation when the same identifier is accessed multiple times within an operation
- Ensures consistency by guaranteeing the same identifier returns the same instance or value throughout the operation lifecycle
- Avoids external dependencies by leveraging language built-in data structures with well-understood performance characteristics
- Provides a simple, predictable caching mechanism that developers can implement without learning a caching library API

Negative:
- In-memory Map caching does not provide persistence, TTL expiration, or distributed cache coherence, limiting applicability to operation-scoped memoization
- Without explicit cache invalidation logic, stale entries may persist for the lifetime of the Map instance, potentially causing inconsistency if underlying resources change
- Memory usage grows linearly with the number of unique identifiers cached, with no automatic eviction mechanism for large datasets
- The pattern requires developers to manually implement cache-check-populate logic at each usage site, increasing boilerplate compared to transparent caching solutions

## Alternatives

- Adopt a third-party caching library with TTL, eviction policies, and cache statistics (rejected)
  Rejected because: The evidence shows simple Map-based caching is sufficient for the observed use cases (operation-scoped memoization of configuration and connection state). Introducing a caching library would add dependency overhead without addressing requirements not present in the evidence.
  When valid: When cache invalidation, TTL expiration, size-based eviction, or cache statistics become requirements
- Implement caching at the I/O layer (file system cache, HTTP client cache) rather than application layer (rejected)
  Rejected because: The evidence shows caching of computed results (resolved agent selections, server entry objects) rather than raw I/O responses. Application-layer caching allows storing derived state that cannot be cached at the I/O layer.
  When valid: When caching raw file contents or HTTP responses without transformation
- Use WeakMap for automatic garbage collection of cached entries (rejected)
  Rejected because: The evidence shows string keys (URLs, directory paths, names) which cannot be used with WeakMap. WeakMap requires object keys and provides no iteration support, limiting applicability.
  When valid: When cache keys are objects and automatic garbage collection of unreferenced entries is required

## Risks

- Memory leaks if Map instances are scoped at module level and accumulate entries without bounds as new identifiers are processed over the application lifetime
  Mitigation: Scope Map instances to operation or request lifecycle where possible. For module-scoped caches, implement periodic clearing or size-based eviction if unbounded growth is observed.
  Owner: engineering team
- Cache inconsistency if underlying resources (configuration files, server state) change while cached entries remain valid, causing stale data to be served
  Mitigation: Document cache lifecycle boundaries clearly. For long-lived caches, implement explicit invalidation when underlying resources change. Consider operation-scoped caching to limit staleness window.
  Owner: engineering team
- Inconsistent caching semantics across modules if each implementation makes different decisions about cache scope, key derivation, or invalidation
  Mitigation: Establish shared utility functions or patterns for common caching scenarios. Document the standard approach in this ADR and reference it during code review.
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
- When implementing cache-check-populate logic, follow the pattern: check cache with get, return if present, compute result if absent, populate cache with set, return result. This ensures consistent cache semantics across the codebase.
- Choose cache scope carefully: operation-scoped (function-local Map) for short-lived caching with automatic cleanup, module-scoped for singleton-like caching where the same instances should be reused across operations. The evidence shows both patterns in use.
- For configuration hierarchies where the same path may be queried multiple times during traversal, cache the resolved result keyed by the canonical path to avoid redundant resolution logic

## Continuation Context


Verify commands:
- Discover the project's static analysis or linting configuration and execute the verification script to ensure Map-based caching follows consistent patterns
- Discover the project's test suite and execute tests covering the modules identified in the evidence to verify cache behavior under repeated identifier access
- Discover the project's code search or grep capability and search for Map instantiation patterns to audit cache scope and key derivation consistency

Accept when:
- All cache layers use native Map instances with set and get methods for storage and retrieval
- Cache keys are derived from stable identifiers (URLs, paths, names) that uniquely identify the cached resource
- Cache-check-populate logic is implemented consistently: check with get, compute if absent, populate with set

## Enforcement

- Verified by: Code review verifying new cache implementations follow the Map-based pattern with appropriate scope and key derivation
- Verified by: Static analysis or linting rules detecting cache implementations that deviate from the established pattern
- Verified by: Unit tests covering cache behavior to ensure consistent semantics across modules
- Violation handling: Code review feedback requesting alignment with the Map-based caching pattern before merge
- Violation handling: Refactoring of non-compliant cache implementations to use Map with appropriate scope and key derivation
- Violation handling: Documentation updates if a legitimate use case requires deviation from the pattern
- Exception process: Document the specific requirement that Map-based caching cannot satisfy (persistence, distributed coherence, TTL expiration, etc.)
- Exception process: Propose an alternative caching approach with rationale for why it is necessary
- Exception process: Obtain approval from the engineering team or technical lead before implementing the alternative
- Exception process: Update this ADR or create a superseding ADR if the exception represents a new architectural direction