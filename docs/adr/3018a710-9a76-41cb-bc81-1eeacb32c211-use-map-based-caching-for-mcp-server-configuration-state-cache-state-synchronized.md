# Use Map-Based Caching for MCP Server Configuration State: Cache State Synchronized

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all configuration management modules handling MCP server state.

## Context

- Configuration management modules (src/vscode/settings.ts, src/mcp/propagateOpenHandsMcp.ts) require in-memory state tracking for MCP server definitions across multiple transport types (SSE, SHTTP, stdio)
- Server configurations are read from external sources (JSON via JSON.parse, TOML via parseTOML) and must be validated, deduplicated, and synchronized before persistence
- Map data structures (existingServerMap, existingSseServers, existingShttpServers, existingStdioServers) are used to cache parsed server configurations keyed by server name or URL
- The caching layer sits between input validation (JSON.parse, parseTOML) and file system persistence operations (fs, fs/promises, FileSystemUtils)
- Concurrent read/write operations (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands) require consistent state management to prevent configuration corruption

## Problem Statement

MCP server configuration management requires a secure, consistent caching mechanism to prevent duplicate entries, ensure atomic updates, and maintain referential integrity across multiple transport protocols while handling untrusted input from JSON and TOML parsers.

## Decision

1. MUST: Cache state MUST be synchronized with file system persistence operations through FileSystemUtils to prevent data loss

## Policy Block

- MUST Cache state MUST be synchronized with file system persistence operations through FileSystemUtils to prevent data loss

In scope:
- MCP server configuration modules (src/vscode/settings.ts, src/mcp/propagateOpenHandsMcp.ts)
- Functions handling server state: readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands, transformRulerToAugmentMcp
- All transport types: SSE (Server-Sent Events), SHTTP (Streaming HTTP), stdio (standard input/output)
- Configuration sources: VSCode settings.json, OpenHands agents.toml

Out of scope:
- Runtime MCP server process management (not configuration state)
- Network transport implementation details
- Authentication and authorization mechanisms
- Logging and monitoring infrastructure

Exceptions:
- EXC-001: Temporary in-memory caching is not required for single-use, ephemeral configuration transformations that do not persist state

## Rationale

- Map data structures provide O(1) lookup and insertion performance with guaranteed key uniqueness, preventing duplicate server configurations that could cause runtime conflicts
- The pattern is observed in 2 files with 91% confidence, demonstrating consistent adoption across configuration management boundaries (VSCode settings, OpenHands TOML)
- Caching parsed configuration state between validation and persistence reduces redundant parsing operations and enables atomic batch updates
- Separation of cache instances by transport type (SSE, SHTTP, stdio) enforces domain isolation and prevents cross-protocol configuration leakage

## Consequences

Positive:
- Guaranteed uniqueness of server configurations through Map key constraints prevents duplicate entries and configuration conflicts
- Atomic Map.set() operations ensure consistent state updates without race conditions in concurrent read/write scenarios
- O(1) lookup performance enables efficient deduplication and existence checks during configuration synchronization
- Clear separation between input validation, caching, and persistence layers improves testability and security boundaries

Negative:
- In-memory Map state is ephemeral and must be rebuilt on each function invocation, increasing memory allocation overhead
- Multiple Map instances per transport type increase memory footprint for large server configurations
- Cache invalidation logic is implicit (rebuild on read) rather than explicit, making cache lifetime reasoning more complex
- No built-in serialization for Map structures requires manual conversion to JSON/TOML-compatible formats for persistence

## Alternatives

- Use plain JavaScript objects with bracket notation for caching server configurations (rejected)
  Rejected because: Plain objects lack guaranteed key uniqueness enforcement, have prototype pollution risks, and provide no type safety for key-value operations
  When valid: Only acceptable for simple, non-security-critical configuration scenarios with trusted input sources
- Implement persistent cache using SQLite or embedded database (rejected)
  Rejected because: Adds external dependency overhead and complexity for configuration state that is already persisted to JSON/TOML files; introduces additional failure modes
  When valid: Valid for high-frequency configuration updates requiring transactional guarantees or complex query patterns
- Use Set data structures for simple existence tracking without full configuration caching (rejected)
  Rejected because: Set structures cannot store associated configuration values (server definitions, URLs, transport parameters) required for synchronization operations
  When valid: Valid for simple deduplication scenarios where only server names/URLs need tracking without full configuration state

## Risks

- Cache state divergence from file system persistence if write operations fail after cache updates
  Mitigation: Implement write-through caching pattern where cache updates only succeed after successful file system persistence; add validation checks on read operations
  Owner: Configuration management module owners
- Memory exhaustion with large server configuration sets due to multiple Map instances per transport type
  Mitigation: Implement configuration size limits and monitoring; consider lazy loading for large configuration sets; add memory profiling to CI pipeline
  Owner: Engineering team
- Input validation bypass if untrusted data is added directly to cache without passing through JSON.parse or parseTOML
  Mitigation: Enforce validation at cache entry points through TypeScript type guards; add runtime assertions; implement code review checklist for cache operations
  Owner: Security review team

## Implementation Notes

- Initialize separate Map instances for each transport type at function scope: const existingSseServers = new Map(); const existingShttpServers = new Map(); const existingStdioServers = new Map();
- Use server name as key for stdio servers (existingStdioServers.set(name, newServer)) and URL as key for SSE/SHTTP servers (existingSseServers.set(url, entry))
- Validate all configuration data through JSON.parse(content) or parseTOML(tomlContent) before Map.set() operations to ensure type safety
- Rebuild cache state from authoritative file system sources (readVSCodeSettings, propagateMcpToOpenHands) rather than maintaining long-lived cache instances
- Use Map.has() for existence checks before updates to implement upsert semantics and prevent unintended overwrites

## Continuation Context


Verify commands:
- grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'
- grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'
- grep -r 'JSON\.parse\|parseTOML' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

Accept when:
- All MCP server configuration modules use Map data structures for caching server state, verified by presence of Map() instantiation and .set() operations
- Input validation (JSON.parse or parseTOML) occurs before all cache insertion operations, verified by code flow analysis
- Separate Map instances exist for each transport type (SSE, SHTTP, stdio) with appropriate key selection (name vs URL)

## Enforcement

- Verified by: Code review checklist requiring Map-based caching for all new configuration management modules
- Verified by: Static analysis rules detecting direct object literal usage for server configuration caching
- Verified by: Unit tests validating cache uniqueness constraints and input validation integration
- Violation handling: CI pipeline fails if grep verification commands do not find expected Map usage patterns
- Violation handling: Code review blocks merge if configuration caching does not follow Map-based pattern
- Violation handling: Runtime assertions log warnings when cache state diverges from file system persistence
- Exception process: Submit exception request to module owner with documented rationale for alternative caching approach
- Exception process: Security review required for any exception involving untrusted input handling
- Exception process: Document approved exceptions in module README with expiration date and migration plan