# Use Map-Based Cache Layer for MCP Server Configuration State: Cache Layer Implementations

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- MCP server configuration management requires reading, transforming, and writing settings across multiple file formats (JSON for VSCode settings, TOML for OpenHands config)
- Configuration operations involve merging new server definitions with existing ones while preserving user-defined servers and avoiding duplicates
- The system must coordinate state between VSCode settings (settings.ts) and OpenHands propagation (propagateOpenHandsMcp.ts), both of which process server collections
- Server definitions are keyed by either name (stdio servers) or URL (SSE/SHTTP servers), requiring consistent lookup and deduplication logic

## Problem Statement

Configuration management functions must efficiently deduplicate, merge, and update MCP server definitions across multiple storage formats without introducing race conditions or data loss when multiple server entries share common identifiers.

## Decision

1. MUST: Cache layer implementations MUST use Map.set() to store entries with their natural identifier (name or URL) as the key

## Policy Block

- MUST Cache layer implementations MUST use Map.set() to store entries with their natural identifier (name or URL) as the key

In scope:
- MCP server configuration management in src/vscode/settings.ts
- OpenHands MCP propagation in src/mcp/propagateOpenHandsMcp.ts
- Operations involving readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands functions
- Server collections keyed by name (stdio) or URL (SSE/SHTTP)

Out of scope:
- Runtime MCP server connection state or session management
- Client-side caching of API responses
- Database or persistent cache layers outside configuration files
- In-memory caching for performance optimization unrelated to configuration deduplication

## Rationale

- Map-based caching provides O(1) lookup and automatic deduplication by key, preventing duplicate server entries when merging configurations
- The pattern appears in both settings.ts and propagateOpenHandsMcp.ts with identical usage (existingServerMap.set, existingSseServers.set, existingShttpServers.set, existingStdioServers.set), indicating a deliberate architectural choice
- Using natural identifiers (name/URL) as Map keys aligns with the domain model where servers are uniquely identified by these properties
- The cache layer isolates the complexity of deduplication from the public API contracts, allowing clean interfaces for configuration operations

## Consequences

Positive:
- Automatic deduplication of server entries by name or URL eliminates manual iteration and comparison logic
- O(1) lookup performance for checking existing servers during merge operations
- Type-safe Map operations reduce runtime errors compared to object property access
- Consistent pattern across multiple configuration modules improves maintainability

Negative:
- Map structures require conversion to/from JSON-serializable formats for file persistence, adding transformation overhead
- Developers must understand the keying strategy (name vs URL) for different server types to use the cache correctly
- Memory overhead of maintaining separate Map instances for each server type (stdio, SSE, SHTTP)
- Pattern is not self-documenting; requires knowledge of the caching strategy to modify configuration logic

## Alternatives

- Use array-based configuration with filter/find operations for deduplication (rejected)
  Rejected because: O(n) lookup performance for each deduplication check would degrade with large server collections, and manual iteration logic is error-prone
  When valid: Valid for very small server collections (< 10 entries) where simplicity outweighs performance
- Use plain objects with bracket notation for caching (e.g., cache[server.name] = server) (rejected)
  Rejected because: Plain objects lack type safety, have prototype pollution risks, and do not distinguish between own properties and inherited ones without hasOwnProperty checks
  When valid: Valid in pure JavaScript environments without TypeScript type checking
- Use Set data structures with custom equality comparators (rejected)
  Rejected because: Sets do not provide key-based lookup; would require iterating the Set to find entries by name/URL, negating performance benefits
  When valid: Valid when only existence checks are needed without retrieval or update operations

## Risks

- Map key collisions if server names or URLs are not properly normalized (e.g., trailing slashes, case sensitivity)
  Mitigation: Implement key normalization functions that canonicalize names and URLs before Map.set operations; add validation tests for common collision scenarios
  Owner: engineering team
- Memory leaks if Map instances are not properly cleared when configuration modules are reloaded or reinitialized
  Mitigation: Ensure Map instances are scoped to function execution contexts (local variables) rather than module-level globals; document lifecycle expectations
  Owner: engineering team
- Inconsistent state if concurrent writes occur to the same configuration file without proper locking
  Mitigation: Implement file-level locking or atomic write operations (write to temp file, then rename); document that configuration functions are not thread-safe
  Owner: engineering team

## Implementation Notes

- When adding new server types, create a dedicated Map instance keyed by the appropriate identifier (name for stdio-like, URL for network-based)
- Always populate the cache Map from existing configuration before performing merge operations to ensure deduplication works correctly
- Convert Map instances to arrays using Array.from(map.values()) before serializing to JSON or TOML formats
- Use TypeScript generics (Map<string, ServerType>) to maintain type safety across the cache layer and prevent type mismatches

## Continuation Context


Verify commands:
- grep -r 'Map<string,' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'
- grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|server\.url|newServer\.name|url|name)'
- npm test -- --grep 'deduplication|cache layer' 2>/dev/null || echo 'No specific cache tests found'

Accept when:
- All configuration modules use Map.set() with appropriate keys (name or URL) for caching server entries
- Grep commands confirm Map<string, ...> declarations and .set() calls in both settings.ts and propagateOpenHandsMcp.ts
- No duplicate server entries appear in output configuration files after merge operations

## Enforcement

- Verified by: Code review checklist requiring Map-based caching for new configuration operations
- Verified by: Static analysis or linting rules detecting array-based deduplication patterns in configuration modules
- Verified by: Integration tests validating no duplicate servers after configuration merge operations
- Violation handling: Code review rejection if new configuration functions use array iteration for deduplication
- Violation handling: CI pipeline failure if integration tests detect duplicate server entries in output files
- Violation handling: Architecture review required for any changes to the cache layer implementation pattern
- Exception process: Document performance or correctness justification for alternative deduplication approach
- Exception process: Obtain approval from architecture review board or senior engineer
- Exception process: Add inline comments explaining why the standard Map-based pattern is not used