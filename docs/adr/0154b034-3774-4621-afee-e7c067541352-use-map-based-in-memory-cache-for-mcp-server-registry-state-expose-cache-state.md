# Use Map-Based In-Memory Cache for MCP Server Registry State: Expose Cache State

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- MCP server configurations are read from multiple sources (TOML files, VSCode settings JSON) and must be propagated to external systems like OpenHands
- Server registry state includes multiple transport types (SSE, SHTTP, STDIO) that require distinct storage and lookup mechanisms
- The system must maintain existing server entries to avoid duplicate registrations and enable idempotent configuration updates
- Public API contracts (propagateMcpToOpenHands, readVSCodeSettings, writeVSCodeSettings, transformRulerToAugmentMcp) require consistent access to server state across file system operations

## Problem Statement

When integrating MCP server configurations with external APIs and managing multiple transport protocols, the system needs a lightweight, in-process mechanism to track existing server registrations, prevent duplicates, and enable efficient lookups by name or URL without introducing external dependencies or persistence overhead.

## Decision

1. MAY: Expose cache state through public API contracts for external system integration

## Policy Block

- MAY Expose cache state through public API contracts for external system integration

In scope:
- MCP server registry operations in src/mcp/propagateOpenHandsMcp.ts
- VSCode settings management in src/vscode/settings.ts
- Public API contracts: propagateMcpToOpenHands, readVSCodeSettings, writeVSCodeSettings, transformRulerToAugmentMcp
- Transport types: SSE, SHTTP, STDIO

Out of scope:
- Persistent storage or database-backed caching
- Distributed cache systems or external cache services
- Server runtime state or connection pooling
- Authentication or authorization token caching

## Rationale

- Map provides O(1) lookup performance for duplicate detection and server retrieval without external dependencies
- Separate Map instances per transport type prevent key collisions between name-based (STDIO) and URL-based (SSE/SHTTP) indexing schemes
- In-memory caching aligns with the synchronous, file-system-driven configuration workflow observed in both propagateOpenHandsMcp.ts and settings.ts
- The pattern supports idempotent configuration updates required by public API contracts that read, transform, and write server configurations

## Consequences

Positive:
- Zero external dependencies for cache layer reduces deployment complexity and startup time
- Fast duplicate detection prevents configuration errors when propagating to external systems
- Type-safe Map operations with TypeScript provide compile-time guarantees for cache access patterns
- Lightweight memory footprint suitable for typical MCP server registry sizes (dozens to hundreds of entries)

Negative:
- In-memory state is lost on process restart, requiring full configuration reload from file system
- No built-in cache invalidation or TTL mechanisms for stale entries
- Memory usage grows linearly with server count without eviction policies
- Concurrent access from multiple processes would require external coordination not provided by Map

## Alternatives

- Use plain JavaScript objects with bracket notation for cache storage (rejected)
  Rejected because: Map provides better type safety, has explicit size property, and avoids prototype pollution risks inherent in plain objects used as dictionaries
  When valid: When targeting environments without ES6 Map support or when serialization to JSON is required
- Implement persistent cache using SQLite or file-based storage (rejected)
  Rejected because: Adds external dependency and I/O overhead for state that is already sourced from configuration files; no evidence of need for persistence beyond file system
  When valid: When server registry state must survive process restarts or be shared across multiple processes
- Use WeakMap for automatic garbage collection of unused server entries (rejected)
  Rejected because: WeakMap requires object keys and does not support string-based indexing by name or URL; no evidence of memory pressure requiring automatic cleanup
  When valid: When cache keys are objects with independent lifecycles and automatic cleanup is required

## Risks

- Memory leak if server entries are added but never removed during long-running processes
  Mitigation: Implement periodic cache clearing or size limits; monitor memory usage in production; document cache lifecycle in API contracts
  Owner: engineering team
- Race conditions if multiple concurrent operations modify the same Map instance without synchronization
  Mitigation: Document single-threaded assumption; use async/await patterns to serialize file system operations; consider mutex if concurrency is introduced
  Owner: engineering team
- Cache inconsistency if configuration files are modified externally without triggering reload
  Mitigation: Implement file system watchers (fs.watch) to detect external changes; provide explicit cache invalidation API; document manual reload procedures
  Owner: engineering team

## Implementation Notes

- Initialize Map instances at module scope (existingSseServers, existingShttpServers, existingStdioServers) to maintain state across function calls
- Use Map.set() for insertions and Map.has() for duplicate checks before adding new entries
- For STDIO servers, use server.name as the key; for SSE/SHTTP servers, use serverDef.url as the key
- Parse configuration files using @iarna/toml for TOML and JSON.parse for JSON before populating cache
- Consider exposing Map.size and Map.clear() through public API for observability and testing

## Continuation Context


Verify commands:
- grep -r 'existingSseServers.set\|existingShttpServers.set\|existingStdioServers.set' src/
- grep -r 'new Map<' src/ | grep -E '(Sse|Shttp|Stdio)Server'
- grep -r '\.has(' src/ | grep -E 'existing(Sse|Shttp|Stdio)Servers'

Accept when:
- All MCP server registry modules use Map.set() to store server entries indexed by name or URL
- Duplicate detection logic uses Map.has() before inserting new entries
- Separate Map instances exist for each transport type (SSE, SHTTP, STDIO) with appropriate key types

## Enforcement

- Verified by: Code review of MCP server registry implementations
- Verified by: Static analysis to detect Map usage patterns in src/mcp/ and src/vscode/
- Verified by: Unit tests verifying duplicate detection and cache lookup behavior
- Violation handling: Reject pull requests that introduce alternative cache mechanisms without architectural review
- Violation handling: Flag code that uses plain objects for server registry state in code review
- Violation handling: Require migration plan for any persistent cache introduction
- Exception process: Document technical justification for alternative cache mechanism
- Exception process: Obtain approval from architecture review board
- Exception process: Update ADR with exception details and rationale