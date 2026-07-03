# Use Map-Based In-Memory Cache for Server Configuration State: Functions Performing Configuration

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Server configuration management requires frequent lookups and updates during runtime operations, particularly when synchronizing MCP server definitions across VSCode settings and OpenHands TOML configuration files
- The codebase handles multiple server types (SSE, SHTTP, STDIO) with distinct identity keys (name-based for STDIO, URL-based for SSE/SHTTP), requiring efficient keyed access patterns
- Configuration synchronization operations in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts perform repeated set operations on server collections, indicating a need for O(1) insertion and lookup
- The pattern emerged across two independent modules handling different configuration formats (JSON for VSCode, TOML for OpenHands), suggesting a common architectural approach to transient state management

## Problem Statement

Configuration synchronization logic requires efficient storage and retrieval of server definitions during read-modify-write cycles across multiple configuration file formats. Without a consistent caching strategy, repeated linear searches through arrays would degrade performance and complicate deduplication logic when merging server configurations from different sources.

## Decision

1. SHOULD: Functions performing configuration synchronization SHOULD construct Maps from existing configuration arrays at the start of the operation and convert back to arrays before serialization

## Policy Block

- SHOULD Functions performing configuration synchronization SHOULD construct Maps from existing configuration arrays at the start of the operation and convert back to arrays before serialization

In scope:
- Server configuration read-modify-write operations in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Transient state management during configuration file synchronization
- Deduplication logic for MCP server definitions across multiple configuration sources
- Runtime operations involving frequent server lookups by name or URL

Out of scope:
- Persistent storage or long-lived caching of server configurations beyond single function invocations
- Database-backed or distributed caching mechanisms
- Client-side browser caching or HTTP response caching
- Configuration state shared across multiple concurrent operations or processes

## Rationale

- Map data structures provide O(1) average-case complexity for insertion and lookup operations, which is optimal for the repeated set operations observed in existingServerMap.set(), existingSseServers.set(), and existingShttpServers.set() calls
- The pattern appears consistently across two independent modules (settings.ts and propagateOpenHandsMcp.ts) handling different configuration formats, indicating an emergent architectural preference rather than isolated implementation choice
- Function-scoped Map instances enable transactional semantics for configuration updates, preventing race conditions and ensuring atomic read-modify-write cycles without requiring external locking mechanisms
- The evidence shows 91% confidence across 2 files with distinct server type handling (STDIO vs SSE/SHTTP), demonstrating the pattern's applicability across heterogeneous configuration schemas

## Consequences

Positive:
- O(1) lookup and insertion performance eliminates linear search overhead when synchronizing server configurations across multiple sources
- Type-safe separation of server collections (by Map instance) enforces correct key usage (name vs URL) at the data structure level
- Function-scoped cache lifetime provides automatic memory cleanup and prevents stale state accumulation
- Simplified deduplication logic through Map's built-in key uniqueness guarantees

Negative:
- Additional memory allocation for Map instances during each configuration operation, though scoped lifetime limits total footprint
- Conversion overhead between array-based serialization formats (JSON/TOML) and Map-based runtime representation
- Increased code complexity from maintaining separate Map instances for different server types
- No persistence mechanism means configuration state must be rebuilt from file system on each operation

## Alternatives

- Use plain JavaScript objects with bracket notation for keyed access instead of Map instances (rejected)
  Rejected because: Plain objects lack type safety for non-string keys, have prototype pollution risks, and provide less explicit semantics for cache operations compared to Map's dedicated API
  When valid: When interoperating with legacy code that expects plain objects or when serialization to JSON is required without conversion
- Maintain server configurations as arrays and use Array.find() for lookups (rejected)
  Rejected because: O(n) lookup complexity degrades performance for repeated operations, and manual deduplication logic is error-prone compared to Map's built-in key uniqueness
  When valid: When server counts are guaranteed to remain small (< 10 entries) and lookup frequency is minimal
- Implement a persistent cache layer with LRU eviction backed by a database or file system (rejected)
  Rejected because: Adds significant complexity and external dependencies for a use case where configuration files already serve as the source of truth and operations complete within milliseconds
  When valid: When configuration synchronization becomes a performance bottleneck due to high frequency or large configuration sizes exceeding memory constraints

## Risks

- Memory pressure if configuration operations are invoked concurrently with large server collections, as each invocation allocates separate Map instances
  Mitigation: Monitor memory usage in production and implement request throttling or connection pooling if concurrent configuration operations exceed available memory
  Owner: engineering team
- Inconsistent key usage across different server types (name vs URL) could lead to cache misses or incorrect deduplication if not properly enforced
  Mitigation: Use TypeScript type guards and separate Map instances per server type to enforce correct key selection at compile time
  Owner: engineering team
- Array-to-Map-to-array conversion overhead may become noticeable if configuration files grow to thousands of server entries
  Mitigation: Establish performance benchmarks for configuration operations and consider alternative data structures or lazy evaluation if conversion overhead exceeds 100ms
  Owner: engineering team

## Implementation Notes

- Initialize Map instances at the start of configuration synchronization functions by iterating over existing server arrays and using appropriate keys (server.name for STDIO, server.url for SSE/SHTTP)
- Use Map.set() for all cache updates during the modification phase, ensuring idempotent behavior for duplicate server definitions
- Convert Map instances back to arrays using Array.from(map.values()) before serializing to JSON or TOML formats
- Consider extracting Map construction and conversion logic into reusable utility functions to reduce code duplication across settings.ts and propagateOpenHandsMcp.ts

## Continuation Context


Verify commands:
- grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'
- grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'
- grep -r 'new Map' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

Accept when:
- Grep commands identify Map instances named existingServerMap, existingSseServers, existingShttpServers, or existingStdioServers in both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Map.set() operations are present in configuration synchronization functions for updating server cache entries
- Map instances are constructed within function scope rather than as module-level globals

## Enforcement

- Verified by: Code review verification that configuration synchronization functions use Map-based caching with appropriate key selection
- Verified by: Static analysis to detect Array.find() usage in hot paths where Map lookups would be more efficient
- Verified by: Performance testing to ensure configuration operations complete within acceptable latency thresholds
- Violation handling: Flag pull requests that introduce linear search patterns in configuration synchronization code paths
- Violation handling: Require performance justification for any deviation from Map-based caching in server configuration operations
- Violation handling: Document exceptions in code comments with rationale for alternative data structure choices
- Exception process: Submit exception request with performance benchmarks demonstrating that alternative approach meets latency requirements
- Exception process: Obtain approval from technical lead for configuration management subsystem
- Exception process: Document exception rationale in ADR amendments or inline code comments