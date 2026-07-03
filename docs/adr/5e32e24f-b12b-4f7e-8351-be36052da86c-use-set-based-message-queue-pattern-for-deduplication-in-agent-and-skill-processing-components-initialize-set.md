# Use Set-Based Message Queue Pattern for Deduplication in Agent and Skill Processing: Components Initialize Set

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent and skill processing components that handle collections requiring deduplication.

## Context

- The codebase processes agent configurations and skill discovery across multiple files and targets, requiring deduplication of keys and paths to prevent redundant operations
- FirebenderAgent and SkillsProcessor both accumulate items (configuration keys and skill targets) that must be unique within their processing contexts
- Set data structures provide O(1) insertion and automatic deduplication, eliminating the need for manual uniqueness checks in collection-building code
- The pattern appears in boundary coordination code where multiple sources contribute to a single collection that must remain unique

## Problem Statement

Without a consistent deduplication strategy, agent and skill processing code risks duplicate entries in collections, leading to redundant file operations, configuration overwrites, and unnecessary processing overhead. Manual array-based deduplication adds complexity and performance overhead.

## Decision

1. SHOULD: Components SHOULD initialize Set collections at the beginning of processing functions that aggregate items from multiple iterations or sources

## Policy Block

- SHOULD Components SHOULD initialize Set collections at the beginning of processing functions that aggregate items from multiple iterations or sources

In scope:
- Agent configuration processing (FirebenderAgent)
- Skill discovery and propagation (SkillsProcessor)
- Any component that aggregates unique items from multiple sources
- Boundary coordination code that prevents duplicate operations

Out of scope:
- Collections where order matters and duplicates are semantically meaningful
- Simple single-source collections with no deduplication requirements
- External API responses that must preserve original structure

## Rationale

- Evidence shows Set.add() usage in both FirebenderAgent (seen.add(key)) and SkillsProcessor (targets.add(target)), indicating a consistent pattern across agent and skill processing boundaries
- Set data structures provide automatic deduplication with O(1) insertion performance, eliminating manual uniqueness checks and reducing code complexity
- The pattern appears specifically at boundaries.message_queues facet, where components coordinate multiple sources into unique collections
- Both files demonstrate paradigm.concurrency_model concerns with multiple async functions, making efficient deduplication critical for performance

## Consequences

Positive:
- Automatic deduplication eliminates entire classes of bugs related to duplicate processing
- O(1) insertion performance improves efficiency in collection-building loops
- Reduced code complexity by removing manual uniqueness checks and array filtering
- Consistent pattern across agent and skill processing improves maintainability

Negative:
- Set data structures do not preserve insertion order in older JavaScript environments (though modern ES6+ guarantees insertion order)
- Requires conversion to arrays when interfacing with APIs expecting array parameters
- Developers unfamiliar with Set API may default to array-based approaches

## Alternatives

- Use arrays with manual deduplication via filter() and includes() checks (rejected)
  Rejected because: O(n) lookup performance for includes() creates quadratic complexity in collection building; more verbose and error-prone than Set-based approach
  When valid: When order preservation is critical and collection size is guaranteed to be small (< 10 items)
- Use Map data structures with dummy values for deduplication (rejected)
  Rejected because: Map adds unnecessary complexity when only key uniqueness is needed; Set is semantically clearer for membership testing
  When valid: When additional metadata must be associated with each unique key
- Use object literals as hash maps for deduplication (rejected)
  Rejected because: Object keys are coerced to strings, losing type information; Set preserves key types and provides cleaner API
  When valid: When targeting pre-ES6 environments without Set support

## Risks

- Developers may inadvertently use arrays for new collection-building code, missing the deduplication pattern
  Mitigation: Add linting rules or code review guidelines to flag array-based collection building in boundary coordination code; document pattern in contribution guidelines
  Owner: engineering team
- Set-to-array conversions may be forgotten when passing collections to external APIs, causing type errors
  Mitigation: Use TypeScript type checking to enforce array types at API boundaries; add runtime validation for critical interfaces
  Owner: engineering team

## Implementation Notes

- Initialize Sets at the beginning of functions that aggregate items: const seen = new Set(); const targets = new Set();
- Use Set.add() in loops or callbacks where items are accumulated: seen.add(key); targets.add(target);
- Convert to arrays when needed for API compatibility: Array.from(targets) or [...targets]
- Consider using Set.has() for efficient membership testing before conditional operations

## Continuation Context


Verify commands:
- grep -r '\.add(' src/agents/ src/core/ | grep -E '(seen|targets|keys|paths)\.add'
- grep -r 'new Set()' src/agents/ src/core/
- npm test -- --grep 'deduplication|unique'

Accept when:
- Set.add() usage is found in FirebenderAgent and SkillsProcessor for collection building
- No manual array deduplication logic (filter + includes) exists in boundary coordination code
- Tests verify that duplicate items are not processed multiple times

## Enforcement

- Verified by: Code review checks for Set usage in new collection-building code
- Verified by: Unit tests verify deduplication behavior in agent and skill processing
- Verified by: Static analysis tools flag array-based deduplication patterns
- Violation handling: Code review feedback requests refactoring to Set-based approach
- Violation handling: Performance regression tests flag quadratic complexity from array-based deduplication
- Violation handling: Bug reports for duplicate processing are traced to missing Set usage
- Exception process: Document specific requirement for array ordering or duplicate preservation
- Exception process: Obtain approval from tech lead for performance-insensitive code paths
- Exception process: Add inline comment explaining why Set-based approach is not applicable