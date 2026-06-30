# Adopt Message Queue Pattern for Subagent Target Collection: Public Contracts Parsedfrontmatter

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all subagent processing workflows.

## Context

- The codebase processes subagent definitions from multiple file sources (markdown, YAML, TOML) requiring aggregation of discovered targets and aliases
- SubagentsUtils.ts and SubagentsProcessor.ts both implement collection patterns using Set.add() operations to accumulate items during file parsing and discovery
- The discovery process involves asynchronous file system operations (fs/promises) and recursive directory traversal requiring incremental result accumulation
- Multiple subagent files may define overlapping or conflicting aliases that must be deduplicated during the collection phase

## Problem Statement

When processing subagent definitions across multiple files and formats, the system needs a consistent pattern for accumulating discovered targets and aliases without duplication, while supporting asynchronous discovery workflows and maintaining clear boundaries between collection and processing phases.

## Decision

1. SHOULD: Public API contracts (ParsedFrontmatter, CopilotToolMapping, getSelectedSubagentTargets) SHOULD expose collected results as immutable data structures

## Policy Block

- SHOULD Public API contracts (ParsedFrontmatter, CopilotToolMapping, getSelectedSubagentTargets) SHOULD expose collected results as immutable data structures

In scope:
- All subagent file parsing operations in SubagentsUtils.ts
- All subagent discovery workflows in SubagentsProcessor.ts
- Functions that aggregate targets, aliases, or mappings from multiple sources
- Asynchronous file system operations that accumulate results incrementally

Out of scope:
- Downstream consumers of collected subagent data (propagateSubagentsForClaude, writeAgentsDirectoryAtomic)
- Validation logic for individual subagent definitions (validateFrontmatter)
- File format parsing (js-yaml, @iarna/toml) which occurs before collection
- Git ignore path processing (getSubagentsGitignorePaths) which operates on collected results

## Rationale

- The evidence shows consistent use of Set.add() patterns in both SubagentsUtils.ts (mapped.add(alias)) and SubagentsProcessor.ts (targets.add(target)), indicating an established architectural pattern
- Message queue-style accumulation separates the concerns of discovery (finding files), parsing (extracting data), and collection (aggregating results), enabling clearer boundaries between processing stages
- Set-based deduplication prevents duplicate subagent registrations when multiple files reference the same alias or target, which is critical for maintaining consistent tool mappings
- The pattern supports the asynchronous, incremental nature of file system traversal where results arrive over time and must be accumulated without blocking

## Consequences

Positive:
- Automatic deduplication of subagent targets and aliases across multiple file sources without explicit duplicate checking logic
- Clear separation between discovery/parsing phases and consumption phases, improving testability and maintainability
- Support for incremental, asynchronous result accumulation during recursive file system operations
- Consistent pattern across SubagentsUtils and SubagentsProcessor modules reduces cognitive load for developers

Negative:
- Set-based collections lose ordering information which may be relevant for priority or precedence rules
- Additional memory overhead from maintaining intermediate collection structures before final propagation
- Implicit deduplication may mask configuration errors where duplicate definitions should be flagged as warnings
- Pattern requires conversion to arrays or other structures for downstream consumers that need ordered or indexed access

## Alternatives

- Direct array accumulation with explicit duplicate checking using Array.includes() or indexOf() (rejected)
  Rejected because: Requires O(n) lookup for each insertion, degrading performance with large subagent sets and adding complexity to every collection point
  When valid: When ordering must be preserved and the number of subagents is guaranteed to remain small (< 50 items)
- Map-based collection storing additional metadata (file source, timestamp) alongside each target/alias (deferred)
  Rejected because: Not rejected; may be adopted if provenance tracking or conflict resolution requires knowing which file defined each alias
  When valid: When debugging duplicate definitions or implementing last-write-wins semantics based on file modification time
- Stream-based processing using async iterators to avoid intermediate collection structures (rejected)
  Rejected because: Increases complexity significantly and prevents random access to collected results needed by getSelectedSubagentTargets and similar APIs
  When valid: When processing extremely large subagent sets (1000+) where memory constraints are critical

## Risks

- Silent deduplication may hide configuration errors where multiple files unintentionally define the same alias with different implementations
  Mitigation: Add optional logging or warning mode that reports when duplicate aliases are encountered during collection, configurable via environment variable
  Owner: engineering team
- Loss of ordering information may cause non-deterministic behavior if downstream consumers implicitly rely on discovery order
  Mitigation: Document that collection order is not guaranteed; convert Sets to sorted arrays before propagation if deterministic ordering is required
  Owner: engineering team
- Performance degradation if Set operations are performed inside tight loops during recursive file traversal
  Mitigation: Profile collection performance with realistic subagent directory structures; consider batching if individual add() calls become a bottleneck
  Owner: engineering team

## Implementation Notes

- Initialize Set collections at the start of discovery functions (discoverSubagents, loadSubagentFile) before entering async loops
- Use descriptive variable names (targets, aliases, mapped) that clearly indicate the collection is accumulating deduplicated results
- Convert Sets to Arrays using Array.from() or spread operator before returning from public API functions if consumers expect iterable collections
- Consider adding debug logging that reports collection size at key checkpoints (after parsing each file, after completing discovery) to aid troubleshooting

## Continuation Context


Verify commands:
- grep -r 'mapped\.add\|targets\.add' src/core/Subagents*.ts
- grep -r 'new Set<' src/core/Subagents*.ts | wc -l
- npm test -- --testPathPattern='Subagents' --verbose

Accept when:
- All subagent collection operations in SubagentsUtils.ts and SubagentsProcessor.ts use Set.add() or equivalent deduplicating data structures
- Public API functions (getSelectedSubagentTargets, discoverSubagents) return deduplicated collections without requiring caller-side deduplication
- Unit tests verify that duplicate aliases from multiple files result in single entries in collected results

## Enforcement

- Verified by: Code review checklist item requiring Set-based collection for new subagent processing functions
- Verified by: Unit tests that verify deduplication behavior when processing multiple subagent files with overlapping aliases
- Verified by: Static analysis or linting rule flagging Array.push() patterns in subagent collection contexts
- Violation handling: Code review feedback requesting refactor to Set-based collection if array accumulation with manual deduplication is detected
- Violation handling: CI test failures if deduplication tests fail after changes to SubagentsUtils or SubagentsProcessor
- Violation handling: Architecture review required for any PR introducing alternative collection patterns in subagent processing code
- Exception process: Document rationale in code comments if alternative collection structure (Map, Array) is required for specific use case
- Exception process: Obtain approval from tech lead if changing collection pattern in existing public API functions
- Exception process: Add compensating tests that verify the alternative approach maintains deduplication guarantees