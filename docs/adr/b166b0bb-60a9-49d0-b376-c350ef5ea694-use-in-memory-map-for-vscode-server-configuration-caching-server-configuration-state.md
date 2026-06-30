# Use In-Memory Map for VSCode Server Configuration Caching: Server Configuration State

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The VSCode settings module (src/vscode/settings.ts) manages server configuration state that requires fast lookup and modification operations during runtime
- Server configurations are keyed by server name and must support both retrieval and update operations without filesystem I/O overhead on every access
- The module exposes public contracts (VSCodeSettings, AugmentMcpServer, readVSCodeSettings, writeVSCodeSettings, transformRulerToAugmentMcp) that require consistent in-memory state representation
- Configuration data is initially loaded from filesystem using JSON.parse and requires a transient cache layer between disk persistence and application logic

## Problem Statement

VSCode server configuration management requires a data structure that provides O(1) lookup and update operations for server entries keyed by name, while maintaining separation between persistent storage (filesystem JSON) and runtime state manipulation.

## Decision

1. MUST: Server configuration state MUST be cached in an in-memory Map structure keyed by server name

## Policy Block

- MUST Server configuration state MUST be cached in an in-memory Map structure keyed by server name

## Rationale

- Evidence shows explicit use of existingServerMap.set(server.name, server) and existingServerMap.set(newServer.name, newServer), demonstrating Map-based caching with name-based keys
- The module integrates filesystem operations (fs, path) with JSON parsing (JSON.parse(content)), indicating a two-tier storage model where Map serves as the runtime cache
- Public API contracts (readVSCodeSettings, writeVSCodeSettings) require consistent state access patterns that benefit from O(1) Map lookups rather than repeated file I/O
- The pattern supports concurrent read operations through shared Map reference while centralizing write synchronization through writeVSCodeSettings

## Consequences

Positive:
- O(1) lookup and update performance for server configuration access by name
- Clear separation between transient runtime state (Map) and persistent storage (filesystem JSON)
- Reduced filesystem I/O overhead for repeated configuration reads during application runtime
- Type-safe key-value access through Map structure with server name as natural key

Negative:
- Memory overhead proportional to number of server configurations held in Map cache
- Potential cache staleness if external processes modify filesystem JSON without cache invalidation
- Increased complexity in maintaining consistency between Map cache and filesystem persistence layer
- No built-in eviction policy if server configuration count grows unbounded

## Alternatives

- Direct filesystem read/write on every configuration access without caching (rejected)
  Rejected because: Would introduce unacceptable I/O latency for frequent configuration lookups during runtime operations, particularly for read-heavy workloads
  When valid: Valid only for infrequent configuration access patterns or when memory constraints prohibit caching
- Use plain JavaScript object literal as cache instead of Map (rejected)
  Rejected because: Object literals lack type safety for key operations and have prototype pollution risks; Map provides cleaner semantics for key-value caching
  When valid: Valid for legacy codebases without ES6 Map support or when serialization to JSON is frequent
- Implement LRU cache with eviction policy for bounded memory usage (deferred)
  Rejected because: Current evidence shows unbounded Map usage; LRU would add complexity without demonstrated need for eviction
  When valid: Valid if server configuration count grows to memory-constrained levels or access patterns show clear hot/cold data separation

## Risks

- Cache inconsistency if filesystem JSON is modified by external processes without triggering cache reload
  Mitigation: Implement filesystem watch mechanism or cache invalidation strategy; document that writeVSCodeSettings is the canonical write path
  Owner: engineering team
- Memory growth if server configuration count increases unbounded without eviction policy
  Mitigation: Monitor Map size in production; implement size limits or LRU eviction if configuration count exceeds threshold
  Owner: engineering team
- Race conditions if concurrent writeVSCodeSettings calls modify Map and filesystem simultaneously
  Mitigation: Implement write serialization through mutex or queue; ensure atomic update of both Map and filesystem
  Owner: engineering team

## Implementation Notes

- Initialize existingServerMap as new Map() at module load time, populate from readVSCodeSettings on first access
- In writeVSCodeSettings, update Map cache first with set() operation, then persist to filesystem to maintain consistency
- Use server.name as Map key consistently across all set() and get() operations to ensure lookup correctness
- Consider adding cache invalidation method for testing or external filesystem change scenarios

## Continuation Context


Verify commands:
- grep -n 'existingServerMap\.set' src/vscode/settings.ts
- grep -n 'new Map()' src/vscode/settings.ts
- grep -n 'JSON\.parse.*content' src/vscode/settings.ts

Accept when:
- Verification commands confirm existingServerMap.set() calls with server.name as key in src/vscode/settings.ts
- Map initialization is present and populated from JSON.parse operations on filesystem content
- Public API functions (readVSCodeSettings, writeVSCodeSettings) interact with Map cache structure

## Enforcement

- Verified by: Code review verification that server configuration access uses Map cache operations
- Verified by: Static analysis to detect direct filesystem reads bypassing cache layer
- Verified by: Unit tests validating Map.set() and Map.get() operations with server.name keys
- Violation handling: Code review rejection if configuration access bypasses Map cache without justification
- Violation handling: Refactoring required if direct filesystem I/O is used for repeated configuration reads
- Violation handling: Architecture review triggered if alternative caching mechanism is proposed
- Exception process: Document exception rationale if direct filesystem access is required for specific use case
- Exception process: Obtain architecture team approval for alternative caching strategies
- Exception process: Update ADR with amendment if Map-based caching proves insufficient for performance or consistency requirements