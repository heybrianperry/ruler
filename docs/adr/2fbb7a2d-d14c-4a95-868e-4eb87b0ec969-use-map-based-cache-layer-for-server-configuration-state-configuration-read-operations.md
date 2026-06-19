# Use Map-Based Cache Layer for Server Configuration State: Configuration Read Operations

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase manages multiple server configurations (VSCode MCP servers, OpenHands SSE/SHTTP/STDIO servers) that require efficient lookup and update operations by unique identifiers (server name or URL).
- Configuration files are read from disk (JSON for VSCode settings, TOML for OpenHands), parsed into memory, and must be synchronized back to disk after modifications.
- The pattern emerged in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts where server entries are stored and retrieved using Map.set() operations with keys derived from server.name or url properties.
- The architecture requires maintaining existing server state while allowing selective updates and additions without losing unmodified entries during write-back operations.

## Problem Statement

Configuration management modules need a consistent in-memory data structure for tracking server definitions that supports O(1) lookup by identifier, preserves insertion order, and facilitates efficient synchronization between disk-based configuration files and runtime state without data loss during partial updates.

## Decision

1. MUST: Configuration read operations MUST populate the Map cache before performing any modifications to preserve existing entries.

## Policy Block

- MUST Configuration read operations MUST populate the Map cache before performing any modifications to preserve existing entries.

## Rationale

- Map provides O(1) lookup and insertion performance critical for configuration synchronization operations that may process multiple server entries.
- The pattern appears in 2 files with 91% significance, demonstrating consistent adoption across VSCode settings management and OpenHands MCP propagation modules.
- Using Map.set() with server-specific keys (name/url) prevents duplicate entries and enables atomic updates to individual server configurations without full-file rewrites.
- The cache layer decouples in-memory state manipulation from filesystem I/O, allowing batch operations and validation before committing changes to disk.

## Consequences

Positive:
- Efficient O(1) lookup and update operations for server configurations by unique identifier.
- Preservation of existing server entries during partial configuration updates prevents data loss.
- Clear separation between read (parse from disk), modify (Map operations), and write (serialize to disk) phases improves code maintainability.
- Type-safe key-value storage with Map's built-in iteration support simplifies conversion back to configuration file formats.

Negative:
- Additional memory overhead from maintaining parallel Map structures alongside parsed configuration objects.
- Requires careful key selection logic to ensure uniqueness and consistency across read-modify-write cycles.
- Potential for cache-disk inconsistency if write operations fail after Map updates, requiring error handling and rollback logic.
- Increased complexity in configuration modules compared to direct array manipulation or object property access patterns.

## Alternatives

- Use plain JavaScript objects with bracket notation for caching server configurations (rejected)
  Rejected because: Plain objects lack guaranteed iteration order, have prototype pollution risks, and do not provide built-in size tracking or clear key-value semantics that Map offers.
  When valid: Acceptable for simple single-type server configurations with string keys that never conflict with Object.prototype properties.
- Use arrays with Array.find() for server lookup and Array.filter() for updates (rejected)
  Rejected because: Array operations require O(n) linear search for lookups and updates, creating performance bottlenecks as server configuration count grows.
  When valid: Suitable for configurations with fewer than 10 servers where linear search overhead is negligible.
- Use WeakMap for server configuration caching (rejected)
  Rejected because: WeakMap requires object keys and does not support string keys (server names/URLs), and lacks iteration capabilities needed for serialization back to configuration files.
  When valid: Only applicable if server objects themselves (not their names/URLs) serve as cache keys and garbage collection of unused servers is desired.

## Risks

- Cache-disk divergence if write operations fail after Map updates, leaving in-memory state inconsistent with persisted configuration.
  Mitigation: Implement transactional write patterns with atomic file operations (write to temp file, then rename) and error handling that invalidates cache on write failure.
  Owner: engineering team
- Key collision or incorrect key derivation logic could cause server configurations to overwrite each other in the Map cache.
  Mitigation: Enforce strict key derivation rules (name for STDIO, url for SSE/SHTTP) with validation tests and consider adding key uniqueness assertions during Map.set() operations.
  Owner: engineering team
- Memory growth if Map caches are not cleared between configuration reload cycles in long-running processes.
  Mitigation: Implement cache lifecycle management with explicit clear() calls before repopulating from disk, or use scoped Map instances within function boundaries.
  Owner: engineering team

## Implementation Notes

- Initialize Map instances with descriptive names (existingServerMap, existingSseServers, existingStdioServers) before reading configuration files to establish clear cache boundaries.
- Use consistent key derivation: server.name for STDIO servers, serverDef.url or url property for SSE/SHTTP servers as evidenced in the detected pattern.
- Populate Map caches immediately after parsing configuration files (JSON.parse for VSCode, parseTOML for OpenHands) to capture existing state before modifications.
- Convert Map entries back to configuration format using Array.from(map.values()) or map iteration before serializing to JSON/TOML for disk writes.

## Continuation Context


Verify commands:
- grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'
- grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|url|newServer\.name)'
- grep -r 'new Map' src/ | wc -l

Accept when:
- All server configuration modules use Map instances for caching server entries with keys derived from server.name or url properties.
- Map.set() operations are present for all server configuration updates in readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands functions.
- No direct array or object mutations bypass the Map cache layer in configuration synchronization code paths.

## Enforcement

- Verified by: Code review verification that new configuration management code uses Map-based caching patterns.
- Verified by: Automated grep-based checks in CI pipeline scanning for Map usage in configuration modules.
- Verified by: Unit tests validating Map.set() operations preserve existing entries during partial configuration updates.
- Violation handling: Code review feedback requesting refactoring to Map-based cache pattern for configuration state management.
- Violation handling: CI pipeline warnings when configuration modules lack Map usage or use alternative data structures for server caching.
- Violation handling: Architecture review escalation for persistent violations or proposals to deviate from the established pattern.
- Exception process: Document technical justification for alternative caching approach with performance or correctness rationale.
- Exception process: Obtain approval from architecture review board for deviations in new configuration management modules.
- Exception process: Update ADR with approved exception cases and conditions under which alternative patterns are valid.