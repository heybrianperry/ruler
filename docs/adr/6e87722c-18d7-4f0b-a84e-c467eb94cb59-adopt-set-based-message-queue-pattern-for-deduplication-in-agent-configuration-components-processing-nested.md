# Adopt Set-Based Message Queue Pattern for Deduplication in Agent Configuration: Components Processing Nested

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all agent configuration processing, MCP server management, skills propagation, and revert operations within the Ruler system.

## Context

- The Ruler system processes agent configurations across nested directory hierarchies, requiring deduplication of file paths, gitignore entries, and MCP server identifiers to prevent duplicate writes and configuration conflicts.
- Multiple agent types (Claude, Copilot, Windsurf, Firebender) generate overlapping configuration artifacts (.mcp.json, .gitignore entries, skill manifests) that must be tracked uniquely across concurrent processing operations.
- The system propagates skills and MCP configurations through directory trees where the same target path may be encountered multiple times through different traversal paths, necessitating idempotent operations.
- Configuration revert operations must track processed paths to avoid redundant file system operations and ensure atomic rollback behavior across multiple agent configurations.

## Problem Statement

Without a consistent deduplication mechanism, the agent configuration system risks writing duplicate entries to gitignore files, processing the same MCP configuration multiple times, and creating redundant skill propagation artifacts, leading to configuration bloat, non-idempotent operations, and potential race conditions in concurrent agent setup workflows.

## Decision

1. MUST: Components processing nested directory hierarchies MUST maintain separate Set instances per processing context to isolate deduplication scope.

## Policy Block

- MUST Components processing nested directory hierarchies MUST maintain separate Set instances per processing context to isolate deduplication scope.

In scope:
- Agent configuration processors (FirebenderAgent, Claude, Copilot, Windsurf)
- MCP server configuration management (nested and flat modes)
- Skills discovery and propagation (SkillsProcessor)
- Configuration revert operations (revert-engine)
- Gitignore path accumulation and writing

Out of scope:
- Runtime agent execution (only configuration-time processing)
- User-facing CLI input validation
- External MCP server implementations
- Test harness utilities (unless testing deduplication behavior)

Exceptions:
- EXC-001: A component explicitly requires ordered duplicate entries for semantic correctness (e.g., layered configuration overrides)

## Rationale

- The evidence shows consistent use of Set.add() patterns across 4 files (nested-mcp-behavior.test.ts, FirebenderAgent.ts, SkillsProcessor.ts, revert-engine.ts) with 91.30% confidence, indicating an established architectural pattern.
- Set-based deduplication provides O(1) insertion and lookup performance while guaranteeing uniqueness, making it optimal for path and identifier accumulation during tree traversal and configuration merging.
- The pattern appears in critical paths including gitignore management (expectedGitignoreEntries.add), MCP server tracking (seen.add), skills propagation (pathSet.add, targets.add), and revert operations (processedPaths.add), demonstrating cross-cutting architectural significance.
- Using native JavaScript Set eliminates the need for manual deduplication logic, reducing code complexity and preventing subtle bugs from array-based uniqueness checks.

## Consequences

Positive:
- Idempotent configuration operations enable safe retries and concurrent agent setup without configuration corruption
- Reduced gitignore file bloat and cleaner configuration artifacts through automatic deduplication
- Improved performance in nested directory processing by eliminating redundant file system operations
- Simplified code maintenance by using native Set semantics instead of custom deduplication logic

Negative:
- Set data structures do not preserve insertion order in older JavaScript environments (pre-ES2015), though this is not a concern for modern Node.js
- Debugging may be slightly harder as Set contents are not as easily inspectable as Arrays in some logging contexts
- Developers unfamiliar with Set semantics may accidentally convert to Array prematurely, losing deduplication guarantees
- Memory overhead of Set instances in scenarios with very small collections (1-2 items) compared to simple array checks

## Alternatives

- Use Array.includes() or Array.find() for manual deduplication before adding items (rejected)
  Rejected because: O(n) lookup performance degrades with collection size, and manual checks are error-prone and verbose compared to Set's built-in uniqueness guarantee
  When valid: Only for trivial cases with guaranteed small collections (< 5 items) where Set overhead is measurable
- Use Map<string, boolean> to track seen items with explicit true values (rejected)
  Rejected because: Map adds unnecessary value storage overhead when only key uniqueness is needed; Set is semantically clearer for membership testing
  When valid: When additional metadata must be associated with each unique item (e.g., timestamps, counts)
- Implement a custom Deduplicator class wrapping Set with domain-specific methods (deferred)
  Rejected because: Current evidence shows direct Set usage is sufficient; abstraction would add complexity without clear benefit
  When valid: If future requirements demand specialized deduplication logic (e.g., case-insensitive paths, normalization) across multiple components

## Risks

- Developers may inadvertently use Arrays instead of Sets in new agent implementations, breaking deduplication guarantees
  Mitigation: Add ESLint rules or code review checklist items to enforce Set usage in configuration accumulation patterns; provide code templates for new agents
  Owner: Engineering team
- Set-based deduplication may mask legitimate duplicate entries that indicate configuration errors (e.g., same MCP server defined twice with different parameters)
  Mitigation: Implement validation layer that detects and warns on duplicate keys with conflicting values before deduplication; log deduplicated items at debug level
  Owner: Engineering team
- Converting Sets to Arrays at the wrong boundary may cause downstream code to re-introduce duplicates
  Mitigation: Document clear API contracts specifying where deduplication occurs; use TypeScript readonly arrays to signal immutability expectations
  Owner: Engineering team

## Implementation Notes

- Initialize Set instances at the beginning of processing functions (e.g., const pathSet = new Set<string>()) to establish clear deduplication scope
- Use Set.add() immediately upon discovering paths, identifiers, or entries rather than batching additions to minimize duplicate processing windows
- Convert Sets to Arrays using Array.from(set) or [...set] only at function return boundaries or when passing to APIs that require arrays
- For file path deduplication, ensure paths are normalized (e.g., using path.resolve() or path.normalize()) before adding to Sets to prevent false uniqueness from path format variations

## Continuation Context


Verify commands:
- grep -r 'new Set<' src/ tests/ | grep -E '(pathSet|seen|targets|processedPaths|expectedGitignoreEntries)' | wc -l
- grep -r '\.add\(' src/ tests/ | grep -E '(pathSet|seen|targets|processedPaths|expectedGitignoreEntries)' | wc -l
- npm test -- --testNamePattern='nested.*MCP.*propagation' --verbose

Accept when:
- All agent configuration processors use Set data structures for accumulating paths, identifiers, and gitignore entries
- Verification commands show Set.add() patterns in SkillsProcessor, FirebenderAgent, revert-engine, and MCP configuration modules
- Integration tests for nested MCP propagation pass, confirming idempotent gitignore and configuration file writes

## Enforcement

- Verified by: Code review checklist requiring Set usage for all new configuration accumulation logic
- Verified by: Integration tests validating idempotent behavior in nested directory scenarios
- Verified by: Static analysis via grep patterns in CI pipeline checking for Set instantiation in configuration modules
- Violation handling: Code review rejection for PRs introducing Array-based deduplication in configuration processing
- Violation handling: CI pipeline warnings when new agent implementations lack Set-based patterns
- Violation handling: Post-merge remediation tickets for violations discovered in production code
- Exception process: Submit architecture review request documenting why Set-based deduplication is incompatible with the specific use case
- Exception process: Provide evidence that ordering or duplicate semantics are required for correctness
- Exception process: Obtain approval from two senior engineers and document exception rationale in code comments and this ADR