# Use Map-Based In-Memory Cache for Server Configuration State: Server Configuration State

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase manages server configuration state across multiple files (src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts) that require fast lookup and update operations
- Server configurations are identified by unique keys (server names or URLs) and must be retrieved, updated, and persisted without duplicates
- The system reads configuration from disk (JSON and TOML formats), transforms it in memory, and writes it back, requiring an intermediate representation that supports efficient key-based operations
- Map data structures provide O(1) lookup and insertion performance for server entries indexed by name or URL, avoiding linear scans through arrays

## Problem Statement

Server configuration management requires efficient key-based lookup, insertion, and update operations on collections of server entries. Without a structured caching layer, the system would need to perform linear searches through arrays or repeatedly parse configuration files, degrading performance and increasing code complexity when synchronizing server state between different configuration formats.

## Decision

1. MUST: Server configuration state MUST be cached in Map data structures indexed by unique identifiers (server name or URL)

## Policy Block

- MUST Server configuration state MUST be cached in Map data structures indexed by unique identifiers (server name or URL)

## Rationale

- Map-based caching provides O(1) time complexity for server lookup and update operations, as evidenced by existingServerMap.set(), existingSseServers.set(), and existingShttpServers.set() patterns across both files
- The pattern separates in-memory state management from disk I/O operations (fs, fs/promises) and parsing logic (JSON.parse, parseTOML), enabling efficient batch updates before persistence
- Using server-specific identifiers (name for stdio, url for SSE/SHTTP) as Map keys prevents duplicate entries and ensures consistent server identity across configuration operations
- The pattern appears in both VSCode settings management and OpenHands MCP propagation, indicating a cross-cutting architectural concern for server configuration state

## Consequences

Positive:
- O(1) lookup and insertion performance for server configuration entries eliminates the need for linear array searches
- Map-based deduplication automatically prevents duplicate server entries when using consistent key strategies
- Clear separation between in-memory cache operations and disk persistence simplifies transaction-like batch updates
- Type-safe key-value semantics reduce the risk of configuration corruption compared to array manipulation

Negative:
- Map instances consume additional memory compared to direct array manipulation, though this is negligible for typical server configuration counts
- Developers must maintain consistency between Map key selection strategies (name vs. url) across different server transport types
- Cache invalidation logic must be explicitly implemented when configuration files are modified externally
- Conversion between Map and serialization formats (JSON, TOML) adds transformation overhead during read/write operations

## Alternatives

- Use plain arrays with Array.find() for server lookups (rejected)
  Rejected because: Linear O(n) search performance degrades with server count and requires manual duplicate checking logic
  When valid: Only viable for configurations with fewer than 10 servers where performance is not a concern
- Use plain objects with bracket notation for key-based access (rejected)
  Rejected because: Objects lack type-safe iteration methods and have prototype pollution risks; Map provides cleaner semantics for non-string keys
  When valid: Acceptable for simple string-keyed caches where Map API is not available (legacy environments)
- Query configuration files directly on each access without caching (rejected)
  Rejected because: Repeated file I/O and parsing (JSON.parse, parseTOML) for each operation introduces unacceptable latency and complexity
  When valid: Only suitable for infrequent configuration access patterns with strict consistency requirements

## Risks

- Cache becomes stale if external processes modify configuration files without triggering cache invalidation
  Mitigation: Implement file watching or cache TTL mechanisms; document that configuration changes require application restart or explicit reload
  Owner: engineering team
- Inconsistent key selection (name vs. url) across different code paths could lead to duplicate server entries or lookup failures
  Mitigation: Establish clear conventions in code documentation and enforce through code review; consider helper functions that encapsulate key selection logic
  Owner: engineering team
- Memory leaks if Map instances are not properly cleared when server configurations are removed
  Mitigation: Implement explicit Map.delete() operations when servers are removed; add unit tests verifying cache cleanup
  Owner: engineering team

## Implementation Notes

- Initialize Map instances at the beginning of configuration read operations: const existingServerMap = new Map(); then populate via Map.set() during parsing
- Use server.name as the Map key for stdio-based servers and url for SSE/SHTTP servers to maintain consistent identity semantics
- Convert Map to array or object format only during serialization: Array.from(existingServerMap.values()) or Object.fromEntries(existingServerMap)
- Consider extracting Map-based cache logic into reusable utility functions to ensure consistent behavior across src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts

## Continuation Context


Verify commands:
- grep -r 'new Map()' src/ | grep -E '(Server|server)' | wc -l
- grep -r '\.set\(' src/ | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)' | wc -l
- grep -r 'Array\.find\(' src/ | grep -E '(server|Server)' | wc -l

Accept when:
- Map-based cache patterns are detected in at least 2 files handling server configuration (verified by grep showing Map instantiation and .set() operations)
- No linear Array.find() patterns exist for server lookups in configuration management code paths
- All server configuration updates use Map.set() with consistent key strategies (name for stdio, url for SSE/SHTTP)

## Enforcement

- Verified by: Automated grep-based verification commands in CI pipeline checking for Map usage patterns
- Verified by: Code review checklist requiring Map-based caching for new server configuration code
- Verified by: Unit tests validating O(1) lookup performance and duplicate prevention behavior
- Violation handling: CI pipeline fails if Array.find() patterns are introduced in server configuration code paths
- Violation handling: Code review requires justification and architectural approval for alternative caching strategies
- Violation handling: Performance regression tests flag degraded lookup times indicating non-Map implementations
- Exception process: Document performance requirements and server count constraints that justify alternative approaches
- Exception process: Obtain approval from technical lead with evidence that Map-based caching is inappropriate for the specific use case
- Exception process: Add inline comments explaining the exception and link to this ADR for future maintainers