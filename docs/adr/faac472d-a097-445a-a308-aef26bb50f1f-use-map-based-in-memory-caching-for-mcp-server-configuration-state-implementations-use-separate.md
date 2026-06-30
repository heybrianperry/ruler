# Use Map-Based In-Memory Caching for MCP Server Configuration State: Implementations Use Separate

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all MCP server configuration management code paths in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts.

## Context

- MCP server configuration management requires tracking existing server definitions by name or URL to prevent duplicates and enable updates
- Configuration files (VSCode settings.json and OpenHands config.toml) are parsed from disk and must be reconciled with new server definitions
- The codebase uses Map data structures (existingServerMap, existingSseServers, existingShttpServers, existingStdioServers) to cache parsed configuration state during read-modify-write operations
- Both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts implement similar patterns for managing server configuration state with Map.set() operations
- Input validation occurs via JSON.parse() and parseTOML() before populating cache structures, establishing a security boundary between untrusted file content and in-memory state

## Problem Statement

Configuration management code must maintain consistent in-memory representations of MCP server definitions during read-modify-write cycles while preventing duplicate entries and enabling efficient lookups by server name or URL. Without a standardized caching approach, configuration synchronization logic becomes error-prone and vulnerable to race conditions or data corruption.

## Decision

1. MAY: Implementations MAY use separate Map instances for different server transport types (stdio, SSE, HTTP) to enforce type safety

## Policy Block

- MAY Implementations MAY use separate Map instances for different server transport types (stdio, SSE, HTTP) to enforce type safety

In scope:
- MCP server configuration management in src/vscode/settings.ts
- MCP server propagation logic in src/mcp/propagateOpenHandsMcp.ts
- Functions readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands
- All code paths that read, modify, or write MCP server configuration files

Out of scope:
- Runtime MCP server connection state or session management
- Long-lived persistent caches or database-backed storage
- Configuration state shared across process boundaries
- Client-side MCP protocol implementation details

Exceptions:
- EXC-001: Configuration files are known to be small (<100 entries) and performance profiling demonstrates Map overhead is unacceptable

## Rationale

- Map data structures provide O(1) lookup and insertion performance for server configuration keyed by name or URL, preventing duplicate entries efficiently
- The pattern is consistently observed across two independent configuration management modules (VSCode settings and OpenHands propagation), indicating established practice
- Separating cache population from file parsing creates a clear security boundary where validation occurs before state mutation
- Using stable identifiers (name/URL) as Map keys ensures idempotent configuration updates and simplifies merge logic

## Consequences

Positive:
- Efficient O(1) duplicate detection and server lookup during configuration merge operations
- Clear separation between file I/O, parsing/validation, and in-memory state management
- Type-safe cache structures when using separate Maps for different server transport types
- Simplified testing through isolated cache manipulation without file system dependencies

Negative:
- Memory overhead for Map structures during configuration operations, though typically negligible for configuration file sizes
- Potential for cache inconsistency if Map state is not properly synchronized with file writes
- Additional complexity in maintaining separate Map instances for different server types
- No built-in concurrency control if multiple processes attempt simultaneous configuration updates

## Alternatives

- Use plain JavaScript objects with bracket notation for caching server configuration (rejected)
  Rejected because: Objects lack type safety for keys, have prototype pollution risks, and provide no semantic distinction between cache operations and property access
  When valid: Only acceptable for legacy code or environments without ES6 Map support
- Use array-based storage with Array.find() for server lookups (rejected)
  Rejected because: O(n) lookup performance degrades with configuration size and complicates duplicate detection logic
  When valid: Acceptable only for configuration files guaranteed to contain fewer than 10 entries
- Implement a custom ConfigurationCache class with encapsulated Map logic (deferred)
  Rejected because: Adds abstraction overhead without clear evidence of reuse across additional modules beyond the two observed
  When valid: Should be reconsidered if pattern extends to 5+ modules or requires shared cache invalidation logic

## Risks

- Concurrent writes from multiple processes could corrupt configuration files if cache state diverges from disk state
  Mitigation: Implement file locking or atomic write-rename patterns; document single-writer assumption in module headers
  Owner: Engineering team
- Parsing errors in JSON.parse() or parseTOML() could leave cache in inconsistent state if not properly handled
  Mitigation: Wrap all parse operations in try-catch blocks; validate cache state before file writes; add integration tests for malformed input
  Owner: Engineering team
- Memory leaks if Map instances are not properly garbage collected after configuration operations complete
  Mitigation: Ensure Maps are function-scoped and not retained in closures; add memory profiling to CI for configuration-heavy test suites
  Owner: Engineering team

## Implementation Notes

- Initialize Map instances at function scope before parsing configuration files to ensure clean state for each operation
- Use consistent key naming: server.name for stdio servers, server.url or serverDef.url for SSE/HTTP servers
- Wrap JSON.parse() and parseTOML() calls in try-catch blocks with appropriate error handling and logging
- Consider extracting common cache-building logic into shared utility functions if pattern extends to additional configuration modules
- Document cache key selection rationale in code comments to prevent future inconsistencies

## Continuation Context


Verify commands:
- grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'
- grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'
- grep -r 'JSON\.parse\|parseTOML' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

Accept when:
- All grep commands return matches confirming Map usage and .set() operations in both configuration modules
- Parsing functions (JSON.parse, parseTOML) are present before cache population logic
- No direct array-based or object-based caching patterns are found in the specified modules

## Enforcement

- Verified by: Code review checklist requiring Map-based caching for new configuration management code
- Verified by: Static analysis rules detecting array.find() or object property access patterns in configuration modules
- Verified by: Integration tests validating duplicate detection and merge behavior using Map semantics
- Violation handling: Code review rejection for configuration code using non-Map caching approaches without documented exception approval
- Violation handling: CI pipeline warnings for grep-based pattern violations in configuration modules
- Violation handling: Architecture review required for any refactoring that removes Map-based caching
- Exception process: Submit exception request with performance benchmarks or technical constraints justifying alternative approach
- Exception process: Obtain approval from architecture review board with documented rationale
- Exception process: Update ADR with approved exception details and monitoring requirements