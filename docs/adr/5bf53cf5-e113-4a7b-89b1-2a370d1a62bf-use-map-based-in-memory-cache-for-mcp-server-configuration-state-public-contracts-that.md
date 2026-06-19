# Use Map-Based In-Memory Cache for MCP Server Configuration State: Public Contracts That

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- MCP server configuration management requires reading, transforming, and writing settings across multiple file formats (JSON for VSCode settings, TOML for OpenHands config)
- Configuration operations involve deduplication and merging of server entries from existing state with new or updated server definitions
- The system handles three distinct MCP server transport types (SSE, SHTTP, STDIO) that require separate indexing strategies based on URL or name keys
- File I/O operations using fs and fs/promises modules necessitate intermediate state representation before persistence
- The pattern appears in public API contract implementations (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands) that expose configuration management to external consumers

## Problem Statement

Configuration management for MCP servers across multiple file formats and transport types requires efficient deduplication, merging, and lookup operations on server entries before persistence, without introducing external caching dependencies or complex state management libraries.

## Decision

1. SHOULD: Public API contracts that manipulate MCP server configurations SHOULD use the Map-based cache pattern for consistency

## Policy Block

- SHOULD Public API contracts that manipulate MCP server configurations SHOULD use the Map-based cache pattern for consistency

In scope:
- MCP server configuration read/write operations in src/vscode/settings.ts
- MCP server propagation logic in src/mcp/propagateOpenHandsMcp.ts
- Public API functions: readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands, transformRulerToAugmentMcp
- Configuration state derived from JSON.parse() and parseTOML() operations

Out of scope:
- Runtime MCP server connection state or active session management
- Long-lived application-wide caches that persist beyond single configuration operations
- Database-backed or distributed cache implementations
- Configuration state for non-MCP server entities

## Rationale

- Map data structures provide O(1) lookup and upsert operations with native JavaScript support, eliminating external dependencies for transient configuration state
- Keying by server name or URL enables automatic deduplication during configuration merges, preventing duplicate server entries in persisted files
- Function-scoped cache instances align with the stateless nature of configuration read/write operations, avoiding memory leaks and stale state issues
- The pattern is consistently applied across both VSCode settings (JSON) and OpenHands config (TOML) file formats, demonstrating cross-format applicability

## Consequences

Positive:
- Zero external dependencies for configuration state management reduces bundle size and attack surface
- Automatic deduplication through Map.set() semantics prevents configuration file corruption from duplicate entries
- Function-scoped cache lifecycle eliminates memory management complexity and stale state bugs
- Consistent pattern across multiple file formats and transport types improves code maintainability

Negative:
- Map-based caching does not persist across function calls, requiring full file re-reads for each configuration operation
- No built-in cache invalidation or TTL mechanisms if pattern is extended to longer-lived contexts
- Memory usage scales linearly with configuration file size during each operation, though mitigated by function scope
- Pattern does not provide transactional semantics for concurrent configuration modifications

## Alternatives

- Use plain JavaScript objects with bracket notation for caching server configurations (rejected)
  Rejected because: Objects lack Map's guaranteed key iteration order and have prototype pollution risks; Map provides cleaner semantics for key-value caching with any key type
  When valid: When configuration keys are guaranteed to be simple strings and iteration order is not important
- Introduce a persistent application-wide cache layer (e.g., Redis, in-memory LRU cache) for MCP server configurations (rejected)
  Rejected because: Adds external dependencies and complexity for state that is already efficiently managed through file I/O; configuration changes are infrequent enough that file reads are acceptable
  When valid: When configuration read operations become a performance bottleneck or when supporting high-frequency configuration queries
- Use array-based linear search for deduplication during configuration merges (rejected)
  Rejected because: O(n) lookup complexity degrades performance with large server lists; Map's O(1) lookup is more efficient and code is more readable
  When valid: When server configuration lists are guaranteed to remain very small (< 10 entries)

## Risks

- Concurrent modifications to configuration files by multiple processes could result in lost updates due to lack of file locking or transactional semantics
  Mitigation: Implement file-level locking using flock or atomic write-rename patterns; document that configuration APIs are not safe for concurrent use
  Owner: engineering team
- Memory exhaustion if configuration files grow to contain thousands of server entries, as entire config is loaded into Map during each operation
  Mitigation: Implement configuration file size limits and validation; monitor memory usage in production; consider streaming parsers if files exceed reasonable thresholds
  Owner: engineering team
- Pattern extension to longer-lived cache contexts could introduce stale state bugs if cache invalidation is not properly implemented
  Mitigation: Maintain function-scoped cache lifecycle as documented; require explicit design review for any persistent cache implementations
  Owner: engineering team

## Implementation Notes

- Use descriptive Map variable names that indicate transport type and purpose (e.g., existingSseServers, existingStdioServers) to improve code clarity
- Initialize Map instances immediately after parsing configuration files and before any transformation logic to ensure cache is populated
- Use Map.set() for all upsert operations to leverage automatic deduplication; avoid manual existence checks before insertion
- Convert Map back to array or object structure using Array.from(map.values()) or Object.fromEntries() before serialization to JSON/TOML

## Continuation Context


Verify commands:
- grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'
- grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|newServer\.name|url|serverDef\.url)'
- grep -r 'readVSCodeSettings\|writeVSCodeSettings\|propagateMcpToOpenHands' src/ --include='*.ts' -A 20 | grep 'Map'

Accept when:
- All MCP server configuration read/write operations in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts use Map-based caching with appropriate key strategies
- Map.set() operations are present for upserting server entries keyed by name (STDIO) or URL (SSE/SHTTP)
- No persistent or application-wide cache instances are introduced for MCP server configuration state

## Enforcement

- Verified by: Code review verification that new configuration management code follows Map-based caching pattern
- Verified by: Automated grep-based checks in CI pipeline to detect Map usage in configuration modules
- Verified by: Unit tests validating deduplication behavior through Map.set() semantics
- Violation handling: Code review feedback requesting refactor to Map-based pattern for consistency
- Violation handling: CI pipeline warnings if configuration code does not use Map for server state
- Violation handling: Architecture review required for any persistent cache implementations
- Exception process: Document technical justification for alternative caching approach in code comments
- Exception process: Obtain approval from tech lead or architect for deviations from Map-based pattern
- Exception process: Update ADR with new alternative or exception if pattern proves insufficient for specific use case