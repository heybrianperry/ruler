# Directory-Keyed Cache as Service Boundary for Agent Resolution: Populate Cache Set Operations During Configuration

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The system loads hierarchical configurations organized by ruler directory paths, where each directory may define agent selection rules
- Agent resolution from configuration is a potentially expensive operation that should not be repeated for the same directory within a session
- Root and nested configuration entries may reference the same ruler directories, requiring consistent agent selection results across multiple access points
- A service boundary is needed to separate configuration loading concerns from agent resolution logic, enabling independent evolution of each subsystem

## Problem Statement

Without a caching layer at the service boundary between configuration loading and agent resolution, the system would re-resolve agent selections every time a ruler directory is accessed, leading to redundant computation and potential inconsistency when the same directory is referenced from multiple configuration entries. The architecture requires a mechanism to cache resolved agents per directory while maintaining a clear separation between configuration parsing and agent selection logic.

## Decision

1. MUST: Populate the cache using set operations during configuration entry processing, storing the result of agent resolution for each ruler directory

## Policy Block

- MUST Populate the cache using set operations during configuration entry processing, storing the result of agent resolution for each ruler directory

In scope:
- Configuration loading operations that resolve agent selections from ruler directory configurations
- Subsystems that access both root and nested configuration entries requiring consistent agent resolution
- Components that bridge configuration parsing and agent selection logic

Out of scope:
- Agent resolution logic internal to the agent subsystem itself
- Configuration parsing that does not involve agent selection
- Single-access configuration reads where caching provides no performance benefit
- Cross-session or persistent caching requirements

## Rationale

- The pattern eliminates redundant agent resolution operations when the same ruler directory is accessed multiple times during configuration loading, improving performance
- Using directory paths as cache keys creates a natural service boundary that aligns with the hierarchical configuration structure, enabling independent evolution of configuration and agent subsystems
- Map-based caching provides O(1) lookup performance while maintaining simplicity and avoiding external dependencies
- The pattern was detected across 2 files with 0.90 significance, indicating established architectural practice in the configuration loading subsystem

## Consequences

Positive:
- Eliminates redundant agent resolution computations for ruler directories accessed multiple times
- Ensures consistent agent selection results for the same directory across root and nested configuration entries
- Creates clear service boundary between configuration loading and agent resolution, enabling independent testing and evolution
- Provides simple, dependency-free caching mechanism using native Map data structures

Negative:
- Introduces memory overhead proportional to the number of unique ruler directories in a configuration hierarchy
- Cache invalidation strategy is not explicitly defined, potentially causing stale data if configurations change during a session
- Adds complexity to the configuration loading flow with an additional caching layer that must be maintained
- Cache scope and lifetime are implicit, making it difficult to reason about when cached data is valid

## Alternatives

- Re-resolve agents on every access without caching (rejected)
  Rejected because: Would cause redundant computation when the same ruler directory is accessed multiple times, degrading performance in hierarchical configuration scenarios
  When valid: Single-access configuration reads where each ruler directory is accessed exactly once
- Pass resolved agents through function parameters instead of caching (rejected)
  Rejected because: Would tightly couple configuration loading and agent resolution logic, eliminating the service boundary and making independent evolution difficult
  When valid: Simple linear configuration flows without hierarchical or multi-access patterns
- Use memoization decorators or library-based caching (rejected)
  Rejected because: Would introduce external dependencies and additional complexity for a straightforward caching requirement that Map data structures handle natively
  When valid: Complex caching requirements with TTL, eviction policies, or cross-session persistence needs

## Risks

- Cache grows unbounded if configuration loading processes many unique ruler directories without clearing the cache
  Mitigation: Document cache scope and lifetime expectations; consider implementing cache size limits or automatic clearing after configuration loading completes
  Owner: engineering team
- Stale cached data if ruler directory configurations change during a session without cache invalidation
  Mitigation: Define explicit cache invalidation strategy or ensure cache scope is limited to single configuration loading operations
  Owner: engineering team
- Implicit cache behavior makes debugging difficult when agent selection results are unexpected
  Mitigation: Add logging or debugging hooks to track cache hits/misses and cache population events
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
- Initialize the cache Map at the appropriate scope for configuration loading operations, ensuring it is accessible to both cache population and retrieval code paths
- Implement cache population immediately after resolving agents for a ruler directory, before any code attempts to retrieve the cached value
- Use consistent key formatting for ruler directory paths to avoid cache misses due to path normalization differences
- Consider implementing cache statistics or metrics to monitor hit rates and identify opportunities for optimization

## Continuation Context


Verify commands:
- Locate the configuration loading subsystem in the repository and identify the cache Map initialization
- Trace code paths that populate the cache during configuration entry processing to verify set operations use ruler directory paths as keys
- Trace code paths that retrieve cached agent selections to verify get operations use the same key format and implement appropriate fallback handling

Accept when:
- Cache Map is initialized before any configuration loading operations that require agent resolution
- All agent resolution results are stored in the cache using ruler directory paths as keys
- All agent selection retrievals check the cache first before performing resolution
- Cache hits return consistent agent selections for the same ruler directory within a session

## Enforcement

- Verified by: Code review verification that configuration loading code uses the cache Map for all agent resolution operations
- Verified by: Unit tests that verify cache population and retrieval behavior with multiple ruler directories
- Verified by: Integration tests that confirm consistent agent selection for the same directory across root and nested configurations
- Violation handling: Code review feedback requiring refactoring to use the cache layer instead of direct agent resolution
- Violation handling: Test failures when agent resolution is performed without cache participation
- Violation handling: Architecture review escalation for proposed changes that bypass the service boundary
- Exception process: Document the specific use case requiring exception in architecture decision log
- Exception process: Obtain approval from configuration subsystem maintainers
- Exception process: Implement alternative caching strategy or justify why caching is not needed for the specific case