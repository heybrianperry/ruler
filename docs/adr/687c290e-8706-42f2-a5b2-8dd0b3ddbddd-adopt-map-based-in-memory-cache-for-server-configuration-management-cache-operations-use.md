# Adopt Map-Based In-Memory Cache for Server Configuration Management: Cache Operations Use

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase manages server configurations across multiple integration points (VSCode settings and OpenHands MCP) requiring efficient lookup and deduplication of server entries
- Server definitions are read from persistent storage (JSON and TOML files) and must be transformed, validated, and synchronized across different configuration formats
- Multiple server types (SSE, SHTTP, STDIO) require consistent keying strategies where URL-based servers use URL as key and STDIO servers use name as key
- Configuration propagation operations involve reading existing state, merging new entries, and writing back atomically, requiring intermediate state management

## Problem Statement

Server configuration management across VSCode settings and OpenHands MCP requires efficient deduplication, lookup, and merging of server entries during read-transform-write cycles, where linear array operations would introduce O(n²) complexity for duplicate detection and increase the risk of configuration corruption during concurrent updates.

## Decision

1. MUST: Cache operations MUST use Map.set() for insertion/update to ensure O(1) lookup complexity and automatic key-based deduplication

## Policy Block

- MUST Cache operations MUST use Map.set() for insertion/update to ensure O(1) lookup complexity and automatic key-based deduplication

In scope:
- Functions performing server configuration read-transform-write cycles (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands)
- Server definition merging and deduplication logic across VSCode and OpenHands MCP integrations
- Configuration modules handling SSE, SHTTP, and STDIO server types in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts

Out of scope:
- Direct file I/O operations using fs, fs/promises, or FileSystemUtils
- Serialization logic (JSON.stringify, TOML stringification) that converts Maps back to arrays for persistence
- Type definitions in ../types that define server configuration schemas
- Validation logic that operates on individual server objects rather than collections

## Rationale

- Map-based caching provides O(1) lookup and insertion complexity compared to O(n) for array-based approaches, critical for configuration merging operations that would otherwise exhibit O(n²) behavior
- Automatic key-based deduplication through Map.set() prevents duplicate server entries without explicit iteration and comparison logic, reducing code complexity and error surface
- The pattern is consistently applied across both VSCode settings (src/vscode/settings.ts) and OpenHands MCP propagation (src/mcp/propagateOpenHandsMcp.ts) modules, indicating established architectural convention
- Ephemeral Map instances scoped to function execution provide thread-safe intermediate state without introducing shared mutable state or requiring synchronization primitives

## Consequences

Positive:
- Configuration merge operations achieve O(n) complexity instead of O(n²), improving performance for large server lists
- Automatic deduplication through Map keys eliminates entire classes of bugs related to duplicate server entries
- Consistent keying strategy (URL vs name) across server types provides clear mental model for developers
- Ephemeral cache lifetime prevents stale state issues and simplifies reasoning about configuration synchronization

Negative:
- Additional memory allocation for Map structures during configuration operations, though scoped to function lifetime
- Requires conversion between array (persisted format) and Map (cache format) at function boundaries, adding transformation overhead
- Developers must understand and maintain the keying strategy distinction between URL-based and name-based server types
- Map-based approach is not directly serializable, requiring explicit conversion back to arrays before persistence

## Alternatives

- Use array-based operations with Array.find() and Array.filter() for deduplication and lookup (rejected)
  Rejected because: Array.find() introduces O(n) lookup complexity, resulting in O(n²) behavior for merge operations across large server lists, and requires explicit deduplication logic that increases code complexity and error potential
  When valid: Valid only for configuration sets with fewer than 10 servers where performance difference is negligible
- Use Set data structure with custom equality comparison for deduplication (rejected)
  Rejected because: Set requires custom equality functions for object comparison and does not provide key-based lookup, necessitating iteration for retrieval and update operations
  When valid: Valid for simple deduplication scenarios without update requirements
- Implement persistent cache layer with external key-value store (Redis, etc.) (rejected)
  Rejected because: Introduces external dependency and operational complexity for ephemeral cache requirements that are satisfied by in-memory Map structures scoped to function execution
  When valid: Valid if configuration synchronization requires distributed coordination or cross-process cache sharing

## Risks

- Inconsistent keying strategy between server types (URL vs name) could lead to incorrect deduplication or lookup failures if developers apply wrong key type
  Mitigation: Document keying strategy in type definitions and function documentation; consider TypeScript discriminated unions to enforce correct key usage at compile time
  Owner: engineering team
- Map-to-array conversion before persistence could introduce ordering inconsistencies if code relies on specific server ordering
  Mitigation: Explicitly document that server ordering is not guaranteed; if ordering is required, implement explicit sorting after Map-to-array conversion
  Owner: engineering team
- Memory pressure from large Map allocations during configuration operations in resource-constrained environments
  Mitigation: Profile memory usage with realistic configuration sizes; implement streaming or chunked processing if configurations exceed memory thresholds
  Owner: engineering team

## Implementation Notes

- Initialize Map instances at the start of configuration read operations: const existingServerMap = new Map(); then populate from parsed file content
- Use server.name as key for STDIO servers and server.url (or serverDef.url) as key for SSE/SHTTP servers to maintain consistent keying strategy
- Convert Map back to array before serialization: Array.from(existingServerMap.values()) or [...existingServerMap.values()]
- Scope Map variables with 'existing' prefix (existingServerMap, existingSseServers) to clearly indicate they represent current state from persistent storage

## Continuation Context


Verify commands:
- grep -r 'existingServerMap\|existingSseServers\|existingShttpServers\|existingStdioServers' src/ --include='*.ts'
- grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
- grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

Accept when:
- Grep commands identify Map-based cache variables (existingServerMap, existingSseServers, etc.) in both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Map.set() operations are present in configuration merge logic for server deduplication and insertion
- Map initialization (new Map()) occurs at the start of configuration read-transform-write functions

## Enforcement

- Verified by: Code review verification that configuration merge functions use Map-based caching with appropriate keying strategies
- Verified by: Static analysis or linting rules to detect array-based deduplication patterns (Array.find, Array.filter) in configuration modules
- Verified by: Unit tests validating O(1) lookup behavior and correct deduplication across server types
- Violation handling: Code review feedback requiring refactoring of array-based approaches to Map-based caching in configuration modules
- Violation handling: Performance regression tests flagging O(n²) behavior in configuration merge operations
- Violation handling: Architecture review escalation for proposed alternatives that deviate from established Map-based pattern
- Exception process: Document performance analysis demonstrating that array-based approach meets requirements for specific bounded configuration size
- Exception process: Obtain architecture review approval with explicit rationale for deviation from Map-based pattern
- Exception process: Add inline comments explaining exception rationale and configuration size constraints that justify deviation