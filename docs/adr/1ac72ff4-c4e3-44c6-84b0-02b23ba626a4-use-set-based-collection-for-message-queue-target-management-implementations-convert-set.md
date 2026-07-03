# Use Set-Based Collection for Message Queue Target Management: Implementations Convert Set

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all subagent processing and utility modules that manage target collections for message queue boundaries.

## Context

- The SubagentsProcessor and SubagentsUtils modules coordinate subagent discovery, selection, and propagation across the system, requiring efficient collection management for targets and aliases
- Message queue boundary operations use Set data structures (targets.add(target), mapped.add(alias)) to accumulate unique identifiers without duplication
- The codebase integrates file system operations (fs/promises), configuration parsing (js-yaml, @iarna/toml), and path manipulation to discover and load subagent definitions from markdown files
- Concurrent operations (discoverSubagents, listMarkdownFilesRecursive, loadSubagentFile) require thread-safe collection semantics to prevent race conditions during target accumulation

## Problem Statement

When coordinating subagent targets across discovery, selection, and propagation workflows, the system must prevent duplicate entries in message queue boundaries while maintaining efficient lookup and insertion performance during concurrent file system operations and configuration parsing.

## Decision

1. MAY: Implementations MAY convert Set collections to arrays for serialization or external API boundaries

## Policy Block

- MAY Implementations MAY convert Set collections to arrays for serialization or external API boundaries

In scope:
- SubagentsProcessor module (src/core/SubagentsProcessor.ts)
- SubagentsUtils module (src/core/SubagentsUtils.ts)
- All functions that accumulate targets or aliases for message queue boundaries
- Concurrent file system operations that discover or load subagent configurations

Out of scope:
- External API responses that require array serialization
- Database persistence layers where Set semantics are not supported
- Legacy modules that have not yet migrated to Set-based collections
- Test fixtures that explicitly verify duplicate handling behavior

Exceptions:
- EXC-001: Performance profiling demonstrates that Set overhead exceeds 10% of operation time for collections smaller than 10 elements

## Rationale

- The evidence shows explicit Set.add() operations in message queue boundary code (targets.add(target), mapped.add(alias)), indicating intentional use of Set semantics for uniqueness guarantees
- Concurrent operations (discoverSubagents, listMarkdownFilesRecursive, loadSubagentFile) benefit from Set's O(1) insertion and lookup performance compared to array-based duplicate checking
- The separation between SubagentsProcessor and SubagentsUtils demonstrates a clear architectural boundary where Set-based collections provide a consistent contract for target management
- Integration with file system operations and configuration parsing (js-yaml, @iarna/toml) requires deterministic handling of potentially duplicate subagent identifiers discovered across multiple sources

## Consequences

Positive:
- Automatic deduplication of targets and aliases eliminates entire class of bugs related to duplicate message queue entries
- O(1) insertion and lookup performance scales efficiently as the number of discovered subagents grows
- Set semantics provide clear contract for uniqueness, making code intent explicit and reducing cognitive load for maintainers
- Thread-safe iteration and mutation patterns reduce risk of race conditions in concurrent discovery operations

Negative:
- Set data structures do not preserve insertion order in older JavaScript environments (pre-ES2015), potentially affecting deterministic testing
- Serialization to JSON or external APIs requires explicit conversion to arrays, adding transformation overhead
- Debugging and logging Set contents requires additional formatting compared to arrays
- Memory overhead of Set implementation may exceed arrays for very small collections (< 5 elements)

## Alternatives

- Use array-based collections with manual duplicate checking via includes() or indexOf() (rejected)
  Rejected because: O(n) duplicate checking creates performance bottleneck for concurrent operations and increases code complexity with manual deduplication logic
  When valid: Only valid for collections guaranteed to have fewer than 5 elements with no concurrent access
- Use Map data structures with target identifiers as keys and metadata as values (rejected)
  Rejected because: Evidence shows only uniqueness requirements without associated metadata storage, making Map overhead unnecessary
  When valid: Valid when target collections require associated metadata or configuration per target
- Use database-level UNIQUE constraints for target persistence (deferred)
  Rejected because: Evidence shows in-memory collection management without database persistence layer
  When valid: Valid when targets are persisted to database and uniqueness must be enforced across process restarts

## Risks

- Set iteration order may differ across JavaScript engine versions, causing non-deterministic behavior in tests or workflows that depend on ordering
  Mitigation: Convert Sets to sorted arrays at API boundaries where ordering matters; document that internal Set order is not guaranteed
  Owner: Engineering team
- Memory leaks if Set collections are not properly cleared after processing large batches of subagent discoveries
  Mitigation: Implement explicit cleanup in finally blocks; add memory profiling to CI pipeline for long-running operations
  Owner: Engineering team
- Type safety erosion if Set<string> types are not enforced, allowing mixed-type collections
  Mitigation: Use TypeScript generic Set<T> types with strict type checking; add linting rules to prevent Set<any> usage
  Owner: Engineering team

## Implementation Notes

- Initialize Set collections at function scope: const targets = new Set<string>(); to ensure type safety
- Use Array.from(targetSet) or [...targetSet] for serialization to JSON or external API responses
- For debugging, use console.log(Array.from(targets)) or implement custom toString() methods for readable logging
- When migrating legacy array-based code, verify that no logic depends on insertion order or array indexing before converting to Set

## Continuation Context


Verify commands:
- grep -r '\.add(' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts | grep -E '(targets|mapped)'
- grep -r 'new Set' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts
- npm test -- --grep 'SubagentsProcessor|SubagentsUtils' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'

Accept when:
- All message queue boundary operations in SubagentsProcessor and SubagentsUtils use Set.add() for target accumulation
- No array-based duplicate checking logic (includes, indexOf, filter) exists in target collection management code
- Tests verify that duplicate targets are automatically deduplicated without manual intervention

## Enforcement

- Verified by: Code review checklist requiring Set usage for all new target collection code
- Verified by: ESLint custom rule detecting array-based duplicate checking patterns in message queue boundary code
- Verified by: CI pipeline grep verification commands checking for Set.add() patterns in SubagentsProcessor and SubagentsUtils
- Violation handling: CI build fails if grep verification commands do not find expected Set.add() patterns
- Violation handling: Code review blocks merge if array-based collections are introduced without documented exception approval
- Violation handling: Quarterly architecture audits identify violations and create remediation tickets with priority based on collection size
- Exception process: Submit exception request to architecture review board with performance benchmark data or technical constraint documentation
- Exception process: Exception approval requires two senior engineer sign-offs and documented re-evaluation criteria
- Exception process: Approved exceptions must be documented in code comments with EXC-ID reference and expiration date