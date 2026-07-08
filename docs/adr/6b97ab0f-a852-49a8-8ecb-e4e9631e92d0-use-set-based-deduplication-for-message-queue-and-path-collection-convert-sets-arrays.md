# Use Set-Based Deduplication for Message Queue and Path Collection: Convert Sets Arrays

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all code that collects file paths, gitignore entries, or configuration keys where uniqueness must be guaranteed.

## Context

- The codebase processes MCP (Model Context Protocol) configurations across nested directory structures, requiring collection of file paths and server keys without duplication
- Multiple agents (Claude, Copilot, Windsurf) generate configuration files and gitignore entries that must be tracked uniquely across project root, module, and submodule directories
- Skills propagation and configuration management require deduplication of file paths and manifest entries to prevent redundant writes and gitignore pollution
- JSON parsing of existing configuration files occurs repeatedly during config merging, requiring safe handling of duplicate keys and entries

## Problem Statement

Without explicit deduplication mechanisms, configuration processors risk writing duplicate gitignore entries, processing the same file paths multiple times, and accumulating redundant keys in configuration manifests, leading to configuration bloat and potential inconsistencies in multi-directory nested structures.

## Decision

1. MAY: Convert Sets to arrays for final output or iteration when order-independent uniqueness is the primary requirement

## Policy Block

- MAY Convert Sets to arrays for final output or iteration when order-independent uniqueness is the primary requirement

In scope:
- File path collection for gitignore updates
- MCP server key aggregation across nested configurations
- Skills manifest entry tracking
- Configuration backup path generation
- Agent-specific configuration file path resolution

Out of scope:
- Collections where duplicates are semantically meaningful
- Ordered sequences where insertion order must be preserved with duplicates
- Performance-critical hot paths where Set overhead is prohibitive
- External API responses that must preserve duplicate entries

Exceptions:
- EXC-001: Performance profiling demonstrates Set operations create unacceptable overhead in a critical path

## Rationale

- The evidence shows consistent use of Set.add() patterns across three distinct modules (nested MCP tests, FirebenderAgent, SkillsProcessor) for path and key deduplication
- Nested directory structures with per-directory configurations create natural duplication risks that Sets efficiently prevent
- JSON.parse() usage before configuration merging ensures type-safe access to server objects and prevents string-based key collisions
- Set-based deduplication provides O(1) insertion and lookup, scaling efficiently as nested directory depth increases

## Consequences

Positive:
- Gitignore files remain clean without duplicate entries across nested project structures
- Configuration merging operations are idempotent and safe to retry
- Memory usage remains bounded even with deep directory nesting
- Code intent is explicit through Set type declarations

Negative:
- Sets do not preserve insertion order in older JavaScript environments (pre-ES2015 semantics)
- Converting between Sets and arrays adds minor overhead in serialization paths
- Developers must remember to use Set.add() instead of array push() patterns
- Debugging Set contents requires conversion to array or use of Set-specific inspection tools

## Alternatives

- Use array filter with indexOf for deduplication (rejected)
  Rejected because: O(n²) complexity makes this prohibitive for nested directory traversal with many paths
  When valid: Only acceptable for small, fixed-size collections with fewer than 10 items
- Use Map with boolean values to track seen items (rejected)
  Rejected because: Map adds unnecessary value storage overhead when only key uniqueness is needed
  When valid: When additional metadata must be associated with each unique key
- Rely on downstream deduplication in file write operations (rejected)
  Rejected because: Pushes complexity to I/O layer and risks writing duplicate entries before deduplication occurs
  When valid: Never for gitignore or configuration file generation

## Risks

- Developers unfamiliar with Set API may use array methods, bypassing deduplication
  Mitigation: Add ESLint rules to detect array usage in path collection contexts; provide code review checklist
  Owner: Engineering team
- Set equality is reference-based; object deduplication requires custom key generation
  Mitigation: Document that Sets work for primitive paths/strings; use Map with serialized keys for object deduplication
  Owner: Engineering team
- JSON.parse() failures on malformed config files could crash configuration loading
  Mitigation: Wrap JSON.parse() in try-catch blocks as shown in FirebenderAgent pattern; log parse errors and continue with empty config
  Owner: Engineering team

## Implementation Notes

- Initialize Sets at the beginning of functions that traverse nested directories (e.g., const pathSet = new Set<string>())
- Use pathSet.add(path.join(...)) patterns to add normalized paths during traversal
- Convert Sets to arrays for final return values using Array.from(set) or [...set] spread syntax
- Wrap JSON.parse() calls in try-catch blocks when reading existing configuration files, defaulting to empty objects on parse failure
- Use Set.has() to check for duplicates before expensive operations like file I/O

## Continuation Context


Verify commands:
- grep -r 'new Set<' src/ tests/ | grep -E '(path|gitignore|seen|targets)' | wc -l
- grep -r '\.add\(' src/ tests/ | grep -E '(Set|pathSet|seen|targets|expectedGitignoreEntries)' | wc -l
- grep -r 'JSON\.parse' src/ | grep -E '(fs\.readFile|existingContent|content)' | wc -l

Accept when:
- All three grep commands return non-zero counts, confirming Set usage, add() method calls, and JSON.parse() patterns exist
- Code review confirms no array-based path collection in nested directory traversal functions
- Integration tests pass for nested MCP configuration generation without duplicate gitignore entries

## Enforcement

- Verified by: Code review checklist requiring Set usage for path/key collection
- Verified by: Integration tests validating gitignore uniqueness across nested structures
- Verified by: ESLint custom rules detecting array usage in deduplication contexts
- Violation handling: Code review rejection if array-based collection is used for paths or configuration keys
- Violation handling: CI failure if integration tests detect duplicate gitignore entries
- Violation handling: Refactoring ticket created for existing violations discovered during audits
- Exception process: Developer documents performance justification with benchmark data
- Exception process: Tech lead reviews alternative deduplication strategy
- Exception process: Exception and rationale recorded in code comments with EXC-001 reference