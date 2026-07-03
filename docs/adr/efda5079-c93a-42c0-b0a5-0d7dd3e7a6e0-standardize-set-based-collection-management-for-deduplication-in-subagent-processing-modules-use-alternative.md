# Standardize Set-Based Collection Management for Deduplication in Subagent Processing: Modules Use Alternative

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all subagent processing modules that manage collections of targets, aliases, or identifiers requiring deduplication.

## Context

- The subagent processing system in src/core/SubagentsProcessor.ts and src/core/SubagentsUtils.ts manages collections of targets, aliases, and identifiers that must remain unique across discovery and propagation operations
- File system operations (fs/promises) and configuration parsing (js-yaml, @iarna/toml) produce data that may contain duplicate entries when aggregated from multiple sources
- Public API contracts (discoverSubagents, getSelectedSubagentTargets, propagateSubagentsForClaude, loadSubagentFile) expose collection management behavior to consumers expecting deterministic, deduplicated results
- The pattern emerged in modules handling concurrent discovery operations (listMarkdownFilesRecursive, writeAgentsDirectoryAtomic) where duplicate prevention is critical for correctness

## Problem Statement

Without a consistent deduplication strategy, subagent processing modules risk accumulating duplicate targets, aliases, or identifiers when aggregating data from multiple file system sources, configuration files, or concurrent discovery operations, leading to redundant processing, incorrect cardinality in collections, and potential runtime errors in downstream consumers.

## Decision

1. MAY: Modules MAY use alternative deduplication strategies (Map, custom comparators) if Set's reference equality is insufficient for complex object identity

## Policy Block

- MAY Modules MAY use alternative deduplication strategies (Map, custom comparators) if Set's reference equality is insufficient for complex object identity

In scope:
- All modules in src/core/ managing subagent targets, aliases, or identifiers
- Functions using fs/promises, js-yaml, or @iarna/toml for configuration aggregation
- Public API contracts returning collections (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile)
- Concurrent discovery and propagation operations

Out of scope:
- Single-source data retrieval where duplicates are structurally impossible
- Collections explicitly requiring duplicate preservation for semantic reasons
- External library code or third-party modules outside src/core/
- Test fixtures or mock data generators

Exceptions:
- EXC-001: Complex object identity requires custom equality semantics beyond reference equality
- EXC-002: Performance profiling demonstrates Set overhead exceeds duplicate processing cost for specific high-frequency operations

## Rationale

- Evidence from SubagentsProcessor.ts and SubagentsUtils.ts shows consistent use of .add() method calls on collection objects, indicating Set-based deduplication is the established pattern
- The boundaries.message_queues facet detection ('targets.add(target)', 'mapped.add(alias)') confirms Set semantics are used at collection insertion points across multiple modules
- Public API contracts (discoverSubagents, getSelectedSubagentTargets, propagateSubagentsForClaude) require deterministic, deduplicated results to prevent downstream processing errors
- File system operations and YAML/TOML parsing naturally produce overlapping data when scanning multiple sources, making automatic deduplication essential for correctness

## Consequences

Positive:
- Eliminates duplicate processing of subagent targets and aliases, reducing computational overhead and preventing logic errors
- Provides O(1) deduplication at insertion time using Set's native implementation, avoiding expensive post-processing deduplication passes
- Establishes consistent collection management semantics across all subagent processing modules, improving code maintainability
- Prevents cardinality bugs in downstream consumers expecting unique collections from public API contracts

Negative:
- Set data structures use reference equality by default, which may not correctly deduplicate complex objects without custom identity logic
- Requires explicit conversion to arrays at API boundaries, adding minor overhead and potential confusion for developers unfamiliar with Set semantics
- May mask underlying data quality issues where duplicates indicate configuration errors or file system anomalies that should be surfaced
- Increases memory overhead compared to arrays for small collections where duplicates are rare

## Alternatives

- Use array-based collections with post-processing deduplication (Array.from(new Set(array)) or lodash.uniq) (rejected)
  Rejected because: Post-processing deduplication is less efficient (O(n) additional pass) and risks exposing duplicates to intermediate processing steps, violating invariants
  When valid: Acceptable for one-time batch operations where collection size is known to be small (<100 items) and intermediate duplicates are harmless
- Use Map with custom key extraction for complex object deduplication (deferred)
  Rejected because: Not rejected; deferred as an approved exception (EXC-001) for cases requiring custom equality semantics
  When valid: Required when deduplicating objects where identity depends on property values rather than reference equality
- Implement no deduplication and rely on upstream data sources to provide unique values (rejected)
  Rejected because: File system discovery and multi-source configuration aggregation inherently produce duplicates; relying on upstream sources is architecturally fragile and shifts responsibility inappropriately
  When valid: Never valid for multi-source aggregation scenarios

## Risks

- Set's reference equality may fail to deduplicate semantically identical objects with different references, leading to subtle duplicate bugs
  Mitigation: Establish coding standard requiring primitive keys (strings, numbers) for Set-based deduplication, or use Map with explicit key extraction for objects. Add unit tests verifying deduplication behavior for all collection types.
  Owner: Core engineering team
- Developers unfamiliar with Set semantics may incorrectly convert to arrays mid-processing, reintroducing duplicates
  Mitigation: Document Set usage in module-level JSDoc, add ESLint rule detecting premature Set-to-array conversion, provide code review checklist item for collection management patterns
  Owner: Engineering team leads
- Performance degradation for very large collections (>10,000 items) if Set overhead exceeds deduplication benefit
  Mitigation: Establish performance benchmarks for collection sizes encountered in production, implement monitoring for collection cardinality, document exception process (EXC-002) for proven performance issues
  Owner: Performance engineering team

## Implementation Notes

- Initialize collections as Set instances at function entry: 'const targets = new Set<string>()'
- Use .add() for insertion throughout processing: 'targets.add(target)' or 'mapped.add(alias)'
- Convert to arrays only at API boundaries using Array.from() or spread operator: 'return Array.from(targets)' or 'return [...targets]'
- For complex objects, extract a unique primitive key and deduplicate on that key, then use Map to preserve object references: 'const uniqueMap = new Map(items.map(item => [item.id, item]))'
- Add TypeScript type annotations to make Set usage explicit in function signatures: 'function collectTargets(): Set<string>' internally, 'function getTargets(): string[]' for public APIs

## Continuation Context


Verify commands:
- grep -r '\.add(' src/core/Subagents*.ts | grep -E '(targets|mapped|aliases)' # Verify Set.add() pattern usage
- grep -r 'new Set' src/core/Subagents*.ts # Confirm Set initialization in collection management
- npm test -- --grep 'deduplication|unique' # Run tests verifying deduplication behavior

Accept when:
- All collection management functions in src/core/Subagents*.ts use Set data structures for aggregation operations
- Grep verification commands confirm .add() method usage for targets, mapped, and aliases collections
- Unit tests demonstrate that duplicate inputs to discovery and parsing functions produce deduplicated outputs
- Public API contracts return arrays with verified unique elements when tested with duplicate input sources

## Enforcement

- Verified by: Code review checklist requiring Set usage verification for all collection management changes
- Verified by: Unit tests covering deduplication behavior for each public API contract
- Verified by: Static analysis via grep patterns in CI pipeline detecting .add() usage on collection variables
- Violation handling: CI pipeline fails if grep verification commands do not find expected Set patterns in modified files
- Violation handling: Code review blocks merge if collection management uses arrays without documented exception approval
- Violation handling: Runtime monitoring alerts if duplicate processing is detected in production telemetry
- Exception process: Developer documents rationale for exception in code comments and pull request description
- Exception process: Exception requires approval from core team member (EXC-001) or architecture review with benchmarks (EXC-002)
- Exception process: Approved exceptions are recorded in module JSDoc and added to architectural decision log