# Revert-engine expanded to track MCP and sidecar provenance at apply-time: Apply Engine Use Trackmcpgeneratedartifacts Mcp Related

Status: proposed
Date: 2025-01-17
Deciders: AI (signal conversion)

## Context

- The revert-engine previously operated in a stateless manner, rediscovering artifacts to clean up at revert-time through filesystem scanning and heuristics.
- Apply-engine now writes MCP configuration paths, MCP provenance markers, backup files, and agent-specific settings sidecars during apply operations.
- Two new APIs, trackMcpGeneratedArtifacts and trackOptionalGeneratedArtifacts, were introduced to record artifacts written during apply into a generatedPaths collection.
- The architecture has shifted from discovery-at-revert-time to apply-time accounting, where apply records what it touched and revert reads this record for cleanup operations.
- This change enables fine-grained provenance tracking for conditionally generated artifacts and those that may have existed before Ruler modified them.
- The implementation spans commits eff6c6fe, 7dbe2b55, d72de7cc, and f3203fad, affecting core apply-engine.ts, revert-engine.ts, and associated test files.

## Problem Statement

The system requires a reliable mechanism to track all artifacts generated or modified during apply operations to ensure complete and accurate cleanup during revert, while addressing the failure mode where apply crashes mid-run leave incomplete generatedPaths records and orphaned artifacts.

## Decision

1. SHOULD: Apply-engine SHOULD use trackMcpGeneratedArtifacts for MCP-related artifacts and trackOptionalGeneratedArtifacts for conditionally generated artifacts.

## Policy Block

- SHOULD Apply-engine SHOULD use trackMcpGeneratedArtifacts for MCP-related artifacts and trackOptionalGeneratedArtifacts for conditionally generated artifacts.

## Rationale

- Apply-time accounting provides deterministic tracking of what was actually written, eliminating ambiguity about which artifacts belong to Ruler versus pre-existing files.
- Recording provenance at write-time captures the exact state of operations, including conditional generation logic that would be difficult to reconstruct during revert.
- Stateful cleanup enables accurate restoration of backups and removal of sidecars that may not follow predictable naming patterns discoverable through heuristics.
- The generatedPaths collection serves as an audit trail of apply operations, supporting both revert correctness and operational transparency.

## Consequences

Positive:
- Fine-grained provenance tracking enables precise cleanup of MCP configurations, backups, and agent-specific sidecars.
- Revert operations become deterministic and reliable, based on recorded facts rather than discovery heuristics.
- The system can safely handle artifacts that existed before Ruler modifications, avoiding accidental deletion of user files.
- Conditional artifact generation is properly tracked regardless of runtime conditions.

Negative:
- Apply crashes or interruptions leave incomplete generatedPaths records, resulting in orphaned artifacts that revert cannot discover.
- The system introduces state management complexity, requiring careful handling of the generatedPaths collection lifecycle.
- Increased coupling between apply-engine and revert-engine through the shared generatedPaths data structure.
- No fallback mechanism exists for artifacts missing from generatedPaths, making the system brittle to partial failures.

## Alternatives

- Retain discovery-based revert with filesystem scanning and heuristics as the primary cleanup mechanism (rejected)
  Rejected because: Discovery-based approaches cannot reliably distinguish Ruler-generated artifacts from pre-existing files, especially for conditionally generated artifacts and backups. Heuristics are fragile and prone to false positives or negatives.
- Implement hybrid approach with apply-time accounting as primary and discovery fallback for missing entries (deferred)
  When valid: This alternative addresses the orphaned artifact failure mode but was not implemented in the current signal. It remains a viable future enhancement to improve resilience.
- Use write-ahead logging or transactional semantics to ensure generatedPaths completeness even during crashes (deferred)
  When valid: Could be considered as a mitigation strategy for the incomplete generatedPaths failure mode, but requires additional infrastructure for transaction management.

## Risks

- Apply crashes mid-run leave incomplete generatedPaths records, causing orphaned artifacts that revert cannot clean up
  Mitigation: Implement crash recovery mechanisms, consider write-ahead logging, or add discovery fallback for artifacts missing from generatedPaths. Monitor apply operation completion rates and investigate partial failures.
  Owner: Apply-engine maintainers
- Memory or storage constraints if generatedPaths grows unbounded for large apply operations with many artifacts
  Mitigation: Implement pagination or streaming for generatedPaths collection. Monitor collection size and establish limits or cleanup policies for completed operations.
  Owner: Apply-engine maintainers
- Race conditions or concurrency issues if multiple apply operations attempt to modify generatedPaths simultaneously
  Mitigation: Implement proper locking or serialization for generatedPaths updates. Document concurrency constraints and test multi-operation scenarios.
  Owner: Core engine maintainers

## Implementation Notes

- The trackMcpGeneratedArtifacts and trackOptionalGeneratedArtifacts APIs are the primary interfaces for recording artifacts during apply operations.
- Implementation spans src/core/apply-engine.ts and src/core/revert-engine.ts with comprehensive test coverage in tests/unit/core/ and tests/integration/mcp-provenance.test.ts.
- PRs #765, #739, and #763 contain the implementation details and review discussions.
- The generatedPaths collection must be persisted durably to survive process restarts between apply and revert operations.
- Consider implementing idempotent append operations to handle retry scenarios safely.

## References

- src/core/apply-engine.ts
- src/core/revert-engine.ts
- tests/unit/core/apply-engine.test.ts
- tests/unit/core/revert-engine.test.ts
- tests/integration/mcp-provenance.test.ts
- Commit: eff6c6fe14058193f91208362f2834c2cb550c7e
- Commit: 7dbe2b55215eb795c00ecadcf3aed4ed3579855d
- Commit: d72de7ccee64cac85b460c8eca2b5b158f567a9b
- Commit: f3203fadfdab1ed68eb4b72f823de6b923297687
- PR #765
- PR #739
- PR #763
- Head Commit: 0fa2caeef4efaef4c5f8ffbedee8feca4b0b00e2