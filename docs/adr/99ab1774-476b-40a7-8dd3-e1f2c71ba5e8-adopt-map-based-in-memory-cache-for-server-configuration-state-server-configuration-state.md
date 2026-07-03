# Adopt Map-Based In-Memory Cache for Server Configuration State: Server Configuration State

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase manages server configuration state across multiple modules (VSCode settings and OpenHands MCP propagation) that require fast lookup and update operations
- Server definitions are loaded from persistent storage (JSON and TOML files) and must be reconciled with existing in-memory state during read-modify-write cycles
- The system handles three distinct server transport types (SSE, SHTTP, STDIO) that require separate namespace management to prevent key collisions
- Configuration updates must preserve existing server entries while allowing selective additions and modifications based on server identity (name or URL)

## Problem Statement

The system requires a consistent mechanism to maintain transient server configuration state during configuration file read-modify-write operations, enabling efficient lookup by server identity (name or URL), deduplication of entries, and preservation of existing state while applying incremental updates across multiple transport protocols.

## Decision

1. MUST: Server configuration state MUST be cached in Map instances keyed by server identity (name for STDIO, URL for SSE/SHTTP) during configuration read-modify-write operations

## Policy Block

- MUST Server configuration state MUST be cached in Map instances keyed by server identity (name for STDIO, URL for SSE/SHTTP) during configuration read-modify-write operations

## Rationale

- Evidence shows consistent use of Map.set() operations across two independent modules (src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts) for caching server configurations during file I/O operations
- The pattern enables efficient deduplication by server identity, preventing duplicate entries when merging new server definitions with existing configurations
- Map-based caching provides O(1) lookup and insertion performance, critical for configuration reconciliation operations that may process multiple server definitions
- The approach separates transient operational state (in-memory Map) from persistent storage (JSON/TOML files), following a clear read-cache-modify-write pattern

## Consequences

Positive:
- Provides O(1) lookup and insertion performance for server configuration reconciliation operations
- Automatic deduplication through Map key semantics eliminates the need for manual duplicate detection logic
- Clear separation between transient cache state and persistent storage simplifies reasoning about data flow
- Native JavaScript Map API reduces dependency on external caching libraries

Negative:
- In-memory Map instances do not persist across function invocations, requiring cache reconstruction on each configuration operation
- Memory overhead scales linearly with the number of server configurations, though typically negligible for configuration management use cases
- No built-in cache invalidation or TTL mechanisms, relying on function scope for lifecycle management
- Pattern requires careful key selection to ensure correct identity semantics across different server transport types

## Alternatives

- Use array-based storage with Array.find() for lookups and manual duplicate checking (rejected)
  Rejected because: O(n) lookup performance degrades with configuration size and requires explicit duplicate detection logic, increasing code complexity
  When valid: Valid for very small configuration sets (< 10 entries) where linear search overhead is negligible
- Implement persistent cache using external key-value store (Redis, memcached) (rejected)
  Rejected because: Introduces external dependency and operational complexity for transient state that only needs to survive a single function invocation
  When valid: Valid if configuration operations require distributed coordination or cross-process state sharing
- Use plain JavaScript objects with bracket notation for key-value storage (rejected)
  Rejected because: Objects lack native iteration order guarantees and have prototype pollution risks; Map provides cleaner semantics for non-string keys
  When valid: Valid for simple string-keyed caches where iteration order is not important

## Risks

- Key collision between different server types if identity semantics are not correctly enforced (e.g., using URL for STDIO servers or name for SSE servers)
  Mitigation: Maintain separate Map instances per transport type and document key selection requirements in code comments
  Owner: engineering team
- Memory leaks if Map instances are inadvertently captured in closures or global scope beyond their intended lifecycle
  Mitigation: Enforce function-scoped cache instances and conduct code reviews to verify proper scope management
  Owner: engineering team
- Race conditions if multiple concurrent operations attempt to read-modify-write the same configuration file without coordination
  Mitigation: Implement file-level locking or atomic write operations using temporary files and rename semantics
  Owner: engineering team

## Implementation Notes

- Initialize Map instances immediately after parsing configuration files using new Map() to ensure clean state
- Use server.name as the key for STDIO transport servers and server.url (or serverDef.url) as the key for SSE and SHTTP transport servers
- Populate the cache by iterating over existing server arrays and calling existingMap.set(key, server) for each entry
- After applying modifications to the cache, convert Map values back to arrays using Array.from(existingMap.values()) before serializing to storage

## Continuation Context


Verify commands:
- grep -r 'new Map<' src/ --include='*.ts' | grep -E '(Server|server)' | wc -l
- grep -r '\.set\(' src/ --include='*.ts' | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)' | wc -l
- grep -r 'Map.*(name|url).*server' src/ --include='*.ts' -A 5 | grep -c 'set('

Accept when:
- Verification commands confirm at least 2 Map instances are used for server configuration caching across the codebase
- Code review confirms separate Map instances exist for each server transport type (SSE, SHTTP, STDIO) with appropriate key selection
- Manual inspection of configuration management functions shows Map.set() operations occur after file parsing and before modifications

## Enforcement

- Verified by: Code review of configuration management modules to verify Map-based caching pattern adherence
- Verified by: Static analysis using grep or AST-based tools to detect Map instantiation and usage patterns
- Verified by: Unit tests that verify deduplication behavior and correct key selection for each transport type
- Violation handling: Pull requests introducing alternative caching mechanisms must provide justification and performance comparison
- Violation handling: Violations detected in code review result in request for refactoring to align with established pattern
- Violation handling: Existing violations are tracked as technical debt items and prioritized for remediation based on impact
- Exception process: Exception requests must document specific performance or functional requirements that Map-based caching cannot satisfy
- Exception process: Exceptions require approval from module owner and architecture review for cross-cutting concerns
- Exception process: Approved exceptions must be documented in code comments with rationale and alternative approach description