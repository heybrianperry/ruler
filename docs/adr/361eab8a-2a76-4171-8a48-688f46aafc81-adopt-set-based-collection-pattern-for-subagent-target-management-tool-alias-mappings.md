# Adopt Set-Based Collection Pattern for Subagent Target Management: Tool Alias Mappings

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all subagent processing and utility modules that manage target collections or alias mappings.

## Context

- The codebase processes subagent configurations from markdown files with YAML frontmatter, requiring discovery, parsing, and validation of subagent metadata across multiple filesystem locations
- Public API contracts (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile, parseFrontmatter) expose subagent processing capabilities to external consumers, necessitating consistent collection semantics
- Subagent targets and tool aliases require deduplication to prevent duplicate registrations when multiple configuration sources or recursive directory traversals are involved
- The SubagentsProcessor and SubagentsUtils modules coordinate filesystem I/O (fs/promises, path), configuration parsing (js-yaml, @iarna/toml), and collection management in a concurrent processing model

## Problem Statement

When processing subagent configurations through public API contracts that involve recursive filesystem discovery and multi-source aggregation, duplicate targets and aliases can emerge from overlapping directory structures or repeated processing cycles, leading to inconsistent registration state and potential runtime conflicts in message queue boundaries.

## Decision

1. MUST: Tool alias mappings MUST use Set data structures (mapped.add(alias)) to prevent duplicate alias registration

## Policy Block

- MUST Tool alias mappings MUST use Set data structures (mapped.add(alias)) to prevent duplicate alias registration

In scope:
- SubagentsProcessor module (src/core/SubagentsProcessor.ts) and all exported functions
- SubagentsUtils module (src/core/SubagentsUtils.ts) and all exported functions
- Any module that aggregates subagent targets, tool aliases, or gitignore paths
- Public API contracts exposed for external consumption of subagent metadata

Out of scope:
- Internal data structures used after initial collection phase where uniqueness is already guaranteed
- Single-source configuration loading where deduplication is not required
- Test utilities that explicitly require duplicate entries for validation scenarios

Exceptions:
- EXC-001: Test code explicitly validates duplicate detection or error handling behavior

## Rationale

- The evidence shows explicit Set-based collection operations (targets.add(target), mapped.add(alias)) in both SubagentsProcessor and SubagentsUtils modules, indicating intentional deduplication semantics
- Public API contracts (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile) expose subagent processing to external consumers, requiring consistent uniqueness guarantees across module boundaries
- Concurrent processing model (discoverSubagents, listMarkdownFilesRecursive, loadSubagentFile) combined with recursive filesystem traversal creates natural conditions for duplicate entries without Set-based deduplication
- Message queue boundaries (boundaries.message_queues) require stable, unique target registration to prevent routing conflicts or duplicate message delivery

## Consequences

Positive:
- Automatic deduplication eliminates duplicate target registration bugs without explicit uniqueness checks in calling code
- Public API contracts provide stronger guarantees to external consumers, reducing integration complexity
- Set-based semantics align naturally with concurrent processing models where order is not significant
- Reduced cognitive load for developers implementing recursive discovery or multi-source aggregation

Negative:
- Set data structures do not preserve insertion order, which may complicate debugging or deterministic testing scenarios
- Conversion between Set and Array representations adds minor overhead at serialization boundaries
- Implicit deduplication may mask configuration errors where duplicates indicate actual problems (e.g., conflicting definitions)
- Developers unfamiliar with Set semantics may not recognize automatic deduplication behavior

## Alternatives

- Use Array-based collections with explicit deduplication logic (filter, includes, indexOf) (rejected)
  Rejected because: Explicit deduplication is error-prone, requires O(n²) complexity for naive implementations, and increases code complexity across multiple call sites. Set-based approach provides O(1) deduplication automatically.
  When valid: When insertion order must be preserved and is architecturally significant, or when duplicate detection must trigger explicit error handling
- Use Map data structures keyed by target/alias identifiers with metadata values (rejected)
  Rejected because: Evidence shows simple add() operations without associated metadata storage. Map would add unnecessary complexity for pure collection semantics where only uniqueness matters.
  When valid: When each target or alias requires associated metadata (timestamps, source paths, validation state) that must be updated on duplicate detection
- Implement custom collection class with uniqueness validation and order preservation (rejected)
  Rejected because: Over-engineering for the observed use case. Native Set provides sufficient semantics without maintenance burden of custom data structures.
  When valid: When uniqueness, order preservation, and custom validation logic are all required simultaneously

## Risks

- Silent deduplication may hide configuration errors where duplicate entries indicate conflicting subagent definitions with different metadata
  Mitigation: Implement validation in parseFrontmatter and loadSubagentFile to detect and warn on duplicate identifiers with differing configurations before Set insertion
  Owner: Core subagent processing team
- Non-deterministic iteration order in Set structures may cause flaky tests or inconsistent behavior in order-dependent operations
  Mitigation: Convert Sets to sorted arrays at public API boundaries (Array.from(set).sort()) and document that internal processing order is not guaranteed
  Owner: API contract owners
- Performance degradation if Set collections grow very large (thousands of subagents) due to hash table overhead
  Mitigation: Monitor collection sizes in production telemetry and implement pagination or streaming APIs if subagent counts exceed 1000 entries
  Owner: Performance engineering team

## Implementation Notes

- Initialize target and alias collections as Set instances at the beginning of aggregation functions: const targets = new Set<string>()
- Use .add() method for all insertions during recursive discovery and parsing phases to leverage automatic deduplication
- Convert Set to Array at public API boundaries using Array.from(set) or [...set] spread syntax when consumers require array semantics
- Document in JSDoc comments that public API contracts return deduplicated collections and do not preserve discovery order
- Consider adding debug logging when duplicates are detected (set.has() check before add()) to aid troubleshooting without changing semantics

## Continuation Context


Verify commands:
- grep -r 'targets\.add\|mapped\.add' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts
- grep -r 'new Set<' src/core/SubagentsProcessor.ts src/core/SubagentsUtils.ts
- npm test -- --grep 'deduplication|unique' --reporter json | jq '.tests[] | select(.title | contains("duplicate"))'

Accept when:
- All target and alias collection operations in SubagentsProcessor.ts and SubagentsUtils.ts use Set.add() method
- Public API contracts (discoverSubagents, getSelectedSubagentTargets, getSubagentsGitignorePaths) return arrays derived from Set collections
- Test suite includes validation that duplicate targets/aliases from multiple sources result in single collection entries

## Enforcement

- Verified by: Code review checklist requiring Set-based collections for all new subagent aggregation functions
- Verified by: Static analysis linting rule detecting Array-based deduplication patterns (filter/includes) in subagent processing modules
- Verified by: CI pipeline test suite validation that public API contracts return unique collections
- Violation handling: Code review rejection for new Array-based deduplication logic in subagent processing modules
- Violation handling: CI build failure if static analysis detects prohibited deduplication patterns
- Violation handling: Runtime warning logs (non-blocking) if duplicate detection logic is triggered, indicating potential configuration issues
- Exception process: Document architectural justification for exception in ADR amendment or module-level README
- Exception process: Obtain approval from core subagent processing team lead
- Exception process: Add inline code comments referencing exception ID (EXC-001) and justification
- Exception process: Register exception in architectural decision log with review date for periodic reassessment