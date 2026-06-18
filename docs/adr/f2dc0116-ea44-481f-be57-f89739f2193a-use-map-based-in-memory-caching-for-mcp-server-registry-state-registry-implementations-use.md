# Use Map-Based In-Memory Caching for MCP Server Registry State: Registry Implementations Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all MCP server registry and configuration management code.

## Context

- The codebase manages MCP (Model Context Protocol) server configurations across multiple transport types (SSE, SHTTP, STDIO) that require persistent state tracking during runtime operations
- Server registry operations involve reading configuration from filesystem sources (TOML files via '@iarna/toml', JSON via VSCode settings) and maintaining active server instances in memory
- The system must coordinate server lifecycle events including registration, lookup, and updates while preventing duplicate server instances and maintaining referential integrity
- Two distinct integration points exist: propagateOpenHandsMcp.ts handles OpenHands MCP propagation with URL-keyed and name-keyed maps, while settings.ts manages VSCode settings with name-keyed maps

## Problem Statement

Without a consistent in-memory caching strategy for MCP server registry state, the system risks duplicate server instantiation, inconsistent lookup behavior across transport types, and potential race conditions when coordinating configuration updates from filesystem sources with active runtime state.

## Decision

1. MAY: Registry implementations MAY use Map.get() for lookups and Map.has() for existence checks to leverage native Map performance characteristics

## Policy Block

- MAY Registry implementations MAY use Map.get() for lookups and Map.has() for existence checks to leverage native Map performance characteristics

In scope:
- All MCP server registry and configuration management modules
- Functions handling server lifecycle: propagateMcpToOpenHands, readVSCodeSettings, writeVSCodeSettings, transformRulerToAugmentMcp
- Transport-specific server collections for SSE, SHTTP, and STDIO protocols
- Configuration parsing operations using '@iarna/toml' and JSON.parse

Out of scope:
- Persistent storage mechanisms (filesystem writes are separate concerns)
- Network transport implementation details
- Server process lifecycle management beyond registry state
- Authentication and authorization logic

Exceptions:
- EXC-001: Legacy server implementations require alternative data structures for backward compatibility

## Rationale

- Map-based caching provides O(1) lookup performance for server registry operations while maintaining insertion order, critical for coordinating multiple transport types
- The evidence shows consistent use of Map.set() across two independent modules (propagateOpenHandsMcp.ts and settings.ts), indicating an established pattern for state management
- Separating cache instances by transport type (SSE, SHTTP, STDIO) isolates failure domains and prevents cross-transport key collisions
- Input validation through parseTOML and JSON.parse before cache insertion establishes a security boundary between untrusted filesystem data and runtime state

## Consequences

Positive:
- Consistent server lookup behavior across all MCP transport types with predictable O(1) performance characteristics
- Prevention of duplicate server instances through Map's unique key constraint
- Clear separation of concerns between configuration parsing (filesystem I/O) and state management (in-memory cache)
- Native JavaScript Map API provides memory-efficient storage with built-in iteration and existence checking

Negative:
- In-memory-only caching loses all state on process restart, requiring full configuration reload from filesystem
- No built-in cache invalidation strategy for stale entries when configuration files change externally
- Map instances are not thread-safe; concurrent access patterns require additional coordination mechanisms
- Memory footprint grows linearly with server count without eviction policy

## Alternatives

- Use plain JavaScript objects with bracket notation for server registry (rejected)
  Rejected because: Objects lack guaranteed iteration order, have prototype pollution risks, and provide no type safety for key operations. Map provides superior performance for frequent additions/deletions.
  When valid: Only for legacy codebases where Map is unavailable (pre-ES6 environments)
- Implement persistent cache using SQLite or embedded database (rejected)
  Rejected because: Adds complexity and I/O overhead for state that is already persisted in configuration files. The evidence shows no requirement for query capabilities or transactional semantics.
  When valid: When server registry requires complex queries, transactions, or must survive process restarts without filesystem re-parsing
- Use WeakMap for automatic garbage collection of unused server entries (rejected)
  Rejected because: WeakMap requires object keys (strings like URLs and names are not eligible) and prevents intentional iteration over registered servers, which is required by the evidence.
  When valid: When server entries are keyed by object references and automatic cleanup is more important than enumeration

## Risks

- Memory leak if server entries are never removed from Map instances when servers are decommissioned
  Mitigation: Implement explicit cleanup methods that call Map.delete() when servers are removed. Add monitoring for Map size growth over time.
  Owner: Engineering team
- Race conditions if multiple concurrent operations modify the same Map instance without synchronization
  Mitigation: Document Map access patterns as single-threaded. If concurrency is required, wrap Map operations in mutex or use atomic update patterns.
  Owner: Engineering team
- Configuration drift if filesystem changes are not reflected in Map cache, leading to stale server state
  Mitigation: Implement filesystem watchers (fs.watch) to detect configuration changes and trigger cache invalidation. Provide manual refresh API.
  Owner: Engineering team

## Implementation Notes

- Initialize Map instances at module scope or in constructor functions to ensure single registry instance per transport type
- Use descriptive variable names that indicate both the data structure and purpose: existingSseServers, existingShttpServers, existingStdioServers, existingServerMap
- Wrap filesystem parsing operations (parseTOML, JSON.parse) in try-catch blocks and validate structure before Map.set() to prevent invalid state
- Consider implementing helper functions like getOrCreateServer() that encapsulate Map.has() + Map.get() + Map.set() patterns for consistency
- Document the key schema for each Map instance (URL string for SSE/SHTTP, name string for STDIO) in module-level comments

## Continuation Context


Verify commands:
- grep -r 'Map<.*>' src/mcp/ src/vscode/ --include='*.ts' | grep -E '(Server|server)' | wc -l
- grep -r '\.set(' src/mcp/propagateOpenHandsMcp.ts src/vscode/settings.ts | grep -E '(existingSseServers|existingShttpServers|existingStdioServers|existingServerMap)' | wc -l
- grep -r 'parseTOML\|JSON\.parse' src/mcp/propagateOpenHandsMcp.ts src/vscode/settings.ts | wc -l

Accept when:
- All MCP server registry modules use Map instances for in-memory state, verified by grep finding Map type declarations for server collections
- Server registration operations use Map.set() method, verified by grep finding at least 5 occurrences across the two identified files
- Configuration parsing precedes cache operations, verified by presence of parseTOML and JSON.parse calls in the same modules

## Enforcement

- Verified by: Code review checklist requiring Map usage for new server registry implementations
- Verified by: Static analysis rules detecting plain object usage for server collections
- Verified by: Unit tests verifying Map.set() and Map.get() behavior for server lifecycle operations
- Violation handling: Code review rejection for PRs introducing non-Map server registries without approved exception
- Violation handling: Static analysis warnings escalated to errors in CI pipeline
- Violation handling: Refactoring tickets created for existing violations discovered during audits
- Exception process: Submit exception request with technical justification to architecture review board
- Exception process: Provide evidence that Map-based approach is infeasible for specific use case
- Exception process: Document approved exception in ADR amendment with migration plan if applicable