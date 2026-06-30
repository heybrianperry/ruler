# Adopt Message Queue Pattern for Subagent Target Collection: Asynchronous File Loading

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all subagent discovery and processing workflows.

## Context

- The codebase processes subagent definitions from multiple sources (markdown files, YAML frontmatter) requiring coordination between discovery and processing phases
- SubagentsUtils.ts and SubagentsProcessor.ts implement asynchronous file loading and validation operations that must accumulate results across multiple invocations
- The system uses Set-based collection patterns (mapped.add(alias), targets.add(target)) to aggregate discovered subagent targets without duplication
- Integration with external tools (Copilot, Claude) requires propagation of collected subagent metadata through processing pipelines

## Problem Statement

The subagent discovery and processing system must coordinate asynchronous file operations across multiple modules while maintaining a consistent collection of unique targets for downstream integration, requiring a pattern that decouples discovery from consumption and prevents duplicate processing.

## Decision

1. MUST: Asynchronous file loading operations (loadSubagentFile, discoverSubagents) MUST populate shared collection structures for downstream processing

## Policy Block

- MUST Asynchronous file loading operations (loadSubagentFile, discoverSubagents) MUST populate shared collection structures for downstream processing

In scope:
- All subagent discovery operations in SubagentsProcessor.ts
- All subagent file loading and validation in SubagentsUtils.ts
- Copilot tool mapping and alias collection
- Claude agent propagation workflows
- Markdown file discovery and frontmatter parsing

Out of scope:
- Direct file system operations not related to subagent discovery
- Synchronous data structures where uniqueness is not required
- Non-subagent configuration loading
- Runtime agent execution (only discovery and setup phases are in scope)

## Rationale

- The evidence shows consistent use of Set.add() patterns across SubagentsUtils.ts and SubagentsProcessor.ts, indicating an established architectural choice for accumulating unique targets
- Asynchronous file operations (loadSubagentFile, discoverSubagents) require a collection pattern that supports incremental accumulation as files are processed
- Integration with external tools (Copilot, Claude) through propagateSubagentsForClaude demonstrates the need for a well-defined handoff point between discovery and consumption
- The separation between SubagentsUtils and SubagentsProcessor establishes clear boundaries between utility operations and orchestration logic

## Consequences

Positive:
- Guaranteed uniqueness of subagent targets prevents duplicate processing and integration errors
- Asynchronous accumulation pattern supports efficient parallel file loading operations
- Clear separation between discovery and processing phases enables independent testing and modification
- Set-based collections provide O(1) duplicate detection without additional validation logic

Negative:
- Set-based collections do not preserve discovery order, which may complicate debugging or deterministic testing
- Shared mutable collection structures require careful coordination to avoid race conditions in concurrent scenarios
- The pattern introduces coupling between discovery operations and the specific collection interface (Set.add)
- Memory overhead of maintaining complete target collections before propagation may impact performance with large subagent sets

## Alternatives

- Use array-based collections with manual deduplication (rejected)
  Rejected because: Arrays require O(n) duplicate checking and additional validation logic, increasing complexity and reducing performance compared to Set-based uniqueness guarantees
  When valid: When discovery order must be preserved and the number of subagents is small enough that O(n) deduplication is acceptable
- Implement a streaming pipeline with immediate propagation per discovered subagent (rejected)
  Rejected because: Immediate propagation would require external tools to handle incremental updates and duplicate detection, shifting complexity to integration points
  When valid: When downstream consumers require real-time notification of discovered subagents and can efficiently handle incremental updates
- Use a message queue library (e.g., Bull, BullMQ) for formal queue management (deferred)
  Rejected because: Current evidence shows in-memory Set-based collection is sufficient; formal queue infrastructure would add dependency overhead without clear benefit for current scale
  When valid: When subagent discovery must be distributed across multiple processes or when persistence and retry semantics are required

## Risks

- Concurrent modifications to shared Set collections could introduce race conditions if discovery operations run in parallel without synchronization
  Mitigation: Document thread-safety requirements and implement mutex locks or atomic operations if parallel discovery is introduced
  Owner: engineering team
- Loss of discovery order information may complicate debugging when subagent loading failures need to be traced to specific file processing sequences
  Mitigation: Implement separate logging of discovery order or maintain parallel ordered collection for diagnostic purposes
  Owner: engineering team
- Memory pressure from accumulating all targets before propagation could impact performance with very large subagent repositories
  Mitigation: Monitor collection sizes and implement streaming propagation if target counts exceed defined thresholds (e.g., >1000 subagents)
  Owner: engineering team

## Implementation Notes

- Use TypeScript Set<T> with appropriate type parameters for all subagent target collections
- Ensure all discovery functions (discoverSubagents, loadSubagentFile) accept collection references as parameters to enable accumulation
- Document the contract that collection structures must support add() operations and uniqueness guarantees
- Consider implementing a wrapper class around Set if additional metadata (discovery order, timestamps) needs to be tracked alongside targets

## Continuation Context


Verify commands:
- grep -r '\.add(' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts | grep -E '(mapped|targets)'
- grep -r 'Set<' src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts
- npm test -- --testPathPattern='Subagents(Utils|Processor)' --testNamePattern='collection|uniqueness'

Accept when:
- All subagent discovery operations use Set-based collections with .add() accumulation patterns
- No duplicate subagent targets appear in propagated collections to external tools
- Tests verify uniqueness guarantees and proper accumulation across multiple discovery invocations

## Enforcement

- Verified by: Code review verification that new subagent discovery code uses Set-based collections
- Verified by: Unit tests validating uniqueness guarantees and accumulation behavior
- Verified by: Integration tests confirming proper propagation to Copilot and Claude tools
- Violation handling: Pull requests introducing array-based collections without uniqueness guarantees must be revised
- Violation handling: Discovery operations that bypass Set.add() patterns trigger code review feedback
- Violation handling: Failed uniqueness tests block merge until collection patterns are corrected
- Exception process: Document specific technical justification for alternative collection patterns
- Exception process: Demonstrate equivalent uniqueness guarantees through alternative mechanisms
- Exception process: Obtain approval from architecture review for deviations from Set-based pattern