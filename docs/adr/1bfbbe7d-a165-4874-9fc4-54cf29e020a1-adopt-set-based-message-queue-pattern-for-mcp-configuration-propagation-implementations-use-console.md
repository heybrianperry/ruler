# Adopt Set-Based Message Queue Pattern for MCP Configuration Propagation: Implementations Use Console

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent configuration propagation workflows involving MCP server definitions, skill targets, and gitignore path management.

## Context

- The codebase implements nested MCP (Model Context Protocol) configuration propagation across multiple directory levels (root, module, submodule) with per-agent enablement flags
- Agent configuration processors (FirebenderAgent, SkillsProcessor) must track unique targets and paths across multiple invocations to prevent duplicate entries in configuration files and gitignore
- The system uses fs/promises and path modules from Node.js core libraries to perform file I/O operations for reading, parsing, and writing JSON configuration files
- Configuration propagation involves coordinating state across multiple agents (Claude, Copilot, Windsurf) with independent MCP enablement settings at different directory levels
- The pattern emerged from the need to aggregate unique configuration targets while processing hierarchical agent configurations with selective enablement

## Problem Statement

When propagating MCP server configurations, skill targets, and gitignore paths across nested directory structures with multiple agents, the system must prevent duplicate entries while maintaining correct per-directory and per-agent configuration isolation. Without a deduplication mechanism, repeated configuration operations would accumulate redundant entries in JSON configuration files and gitignore, leading to configuration bloat and potential conflicts.

## Decision

1. MAY: Implementations MAY use console.warn for logging parse failures when reading existing configuration files

## Policy Block

- MAY Implementations MAY use console.warn for logging parse failures when reading existing configuration files

In scope:
- MCP server configuration propagation across nested directories
- Gitignore path management for agent-specific configuration files
- Skill target discovery and propagation workflows
- Agent configuration processors (FirebenderAgent, SkillsProcessor, and similar)
- JSON configuration file parsing and serialization operations

Out of scope:
- Runtime message queue implementations for inter-process communication
- Network-based message broker integrations
- Database-backed queue systems
- Event streaming platforms
- Non-configuration-related Set usage patterns

## Rationale

- The evidence shows Set.add() operations in three distinct contexts (expectedGitignoreEntries, seen, targets) across integration tests and agent processors, indicating a consistent pattern for deduplication
- Nested MCP propagation test demonstrates the need to track unique configuration targets across multiple directory levels with selective agent enablement, requiring stateful accumulation without duplicates
- FirebenderAgent and SkillsProcessor both implement configuration propagation functions that must coordinate state across multiple file operations, making Set-based deduplication a natural fit
- The pattern supports the architectural requirement for per-directory, per-agent configuration isolation while preventing duplicate entries in shared resources like gitignore

## Consequences

Positive:
- Eliminates duplicate entries in gitignore files, MCP server configurations, and skill targets across multiple propagation operations
- Provides O(1) lookup and insertion performance for deduplication checks during configuration processing
- Simplifies configuration aggregation logic by leveraging native Set semantics rather than manual array deduplication
- Enables idempotent configuration operations where repeated invocations produce consistent results

Negative:
- Set data structures do not preserve insertion order in older JavaScript environments (pre-ES2015), though this is not a concern for modern Node.js
- Requires conversion to arrays when serializing to JSON or writing to files, adding a transformation step
- Memory overhead of Set objects compared to simple arrays, though negligible for typical configuration sizes
- Developers unfamiliar with Set semantics may attempt to use array methods, leading to runtime errors

## Alternatives

- Use array-based deduplication with filter or includes checks before adding entries (rejected)
  Rejected because: Array-based deduplication has O(n) lookup complexity, leading to O(n²) performance for large configuration sets. The evidence shows Set.add() is already implemented, indicating this alternative was considered and rejected.
  When valid: Only valid for very small configuration sets (< 10 entries) where performance difference is negligible
- Use Map data structures with boolean values to track seen entries (rejected)
  Rejected because: Map requires storing unnecessary boolean values when only key uniqueness matters. Set provides cleaner semantics for membership testing without value overhead.
  When valid: Valid when additional metadata must be associated with each unique entry beyond simple presence
- Implement database-backed deduplication with unique constraints (rejected)
  Rejected because: Introduces external dependency and I/O overhead for what is fundamentally an in-memory configuration processing task. The evidence shows file-based configuration without database integration.
  When valid: Valid for distributed systems where multiple processes must coordinate deduplication across process boundaries

## Risks

- Set-based deduplication may mask configuration errors where duplicate entries indicate a logic bug rather than expected behavior
  Mitigation: Implement logging at debug level when Set.add() encounters an existing entry to aid troubleshooting. Add integration tests that verify expected Set sizes after propagation.
  Owner: Engineering team
- Converting Sets to arrays for JSON serialization may introduce ordering inconsistencies across runs, complicating diff-based configuration validation
  Mitigation: Sort array output after Set conversion before writing to configuration files. Document that configuration file ordering is normalized alphabetically.
  Owner: Engineering team
- Set equality checks require converting to arrays or iterating, making it difficult to assert expected configuration state in tests
  Mitigation: Provide test utility functions for Set comparison (e.g., setsEqual, setContainsAll). Use Array.from() with sorting for snapshot testing.
  Owner: Engineering team

## Implementation Notes

- Initialize Set instances at the beginning of configuration propagation workflows before iterating over agents or directories
- Use Set.add() for all operations that accumulate unique identifiers: gitignore paths, MCP server keys, skill targets, and configuration file paths
- Convert Sets to arrays using Array.from() or spread operator before JSON serialization or file writing operations
- Consider sorting converted arrays to ensure deterministic output ordering for version control and diff comparison
- Implement helper functions like extractServers() to normalize configuration structure before extracting keys for Set-based deduplication

## Continuation Context


Verify commands:
- grep -r 'new Set<' src/ tests/ | grep -E '(gitignore|targets|seen|servers)' | wc -l
- grep -r '\.add\(' src/ tests/ | grep -E '(expectedGitignoreEntries|targets|seen)' | wc -l
- npm test -- --testNamePattern='Nested MCP propagation' --verbose

Accept when:
- Grep commands identify at least 3 distinct Set instantiations and 5+ Set.add() operations across configuration processors
- Nested MCP propagation integration test passes, verifying no duplicate entries in gitignore or MCP configuration files
- Code review confirms all configuration aggregation workflows use Set-based deduplication before writing output

## Enforcement

- Verified by: Integration tests verify Set-based deduplication behavior in nested MCP propagation scenarios
- Verified by: Code review checklist includes verification that configuration processors use Set for accumulation
- Verified by: Static analysis or linting rules flag array-based deduplication patterns in configuration propagation code paths
- Violation handling: CI pipeline fails if integration tests detect duplicate entries in generated configuration files
- Violation handling: Code review blocks merge if configuration processors use array-based deduplication instead of Set
- Violation handling: Runtime warnings logged when duplicate entries are detected during configuration propagation
- Exception process: Document architectural justification for array-based approach in code comments with ADR reference
- Exception process: Obtain approval from architecture review board for performance-critical paths where Set overhead is measured and significant
- Exception process: Add explicit test coverage demonstrating that array-based deduplication maintains correctness guarantees