# Adopt Array.find() for Collection Queries in Test Assertions: Tests Extract Found

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Integration tests require validation of specific entities within collections returned from core modules (allAgents, config.diagnostics)
- Test scenarios involve querying collections to verify presence of specific items matching predicate conditions (agent identifier, diagnostic code)
- The codebase uses Array.find() with predicate functions to locate single matching items from in-memory collections during test execution
- Test assertions depend on retrieving specific entities to validate their properties or existence within larger result sets

## Problem Statement

Integration tests need a consistent, readable pattern for querying in-memory collections to locate specific entities for assertion validation, without introducing external query libraries or complex filtering logic that obscures test intent.

## Decision

1. MAY: Tests MAY extract found entities into named variables when multiple assertions are performed on the same entity

## Policy Block

- MAY Tests MAY extract found entities into named variables when multiple assertions are performed on the same entity

In scope:
- Integration test files using describe/it test frameworks
- Test assertions validating entity presence in collections returned from core modules
- Queries against in-memory arrays of objects with identifying properties

Out of scope:
- Production application code outside test contexts
- Database queries or ORM operations
- Queries requiring multiple matching results (use Array.filter() instead)
- Performance-critical collection operations requiring indexed access

## Rationale

- Evidence shows consistent use of Array.find() with predicate functions in two integration test files (tests/integration/skills-mcp.test.ts, tests/unified-config.mcp-toml-load.test.ts) for querying collections
- The pattern provides readable, inline predicate logic that clearly expresses test intent without external dependencies
- Native JavaScript Array.find() offers sufficient performance for test-time collection queries over small to medium datasets
- The approach maintains consistency with standard JavaScript idioms and requires no additional query abstraction layers

## Consequences

Positive:
- Test code remains readable with inline predicate logic that clearly expresses matching criteria
- No external query library dependencies required, reducing test infrastructure complexity
- Native JavaScript Array methods provide predictable behavior across Node.js versions
- Pattern is easily understood by developers familiar with standard JavaScript array operations

Negative:
- Array.find() performs linear search with O(n) complexity, which may be inefficient for large collections
- No compile-time type safety for predicate functions in plain JavaScript contexts
- Pattern does not scale to complex multi-condition queries without nested logic or multiple find operations
- Undefined results require explicit checking before property access, adding boilerplate to test assertions

## Alternatives

- Use Array.filter() followed by index access [0] to retrieve first matching element (rejected)
  Rejected because: Array.filter() creates intermediate array for all matches, adding unnecessary overhead when only first match is needed; less semantically clear than find() for single-item retrieval
  When valid: When multiple matching items need to be collected and first item selected from filtered results
- Introduce lodash _.find() or similar utility library for collection queries (rejected)
  Rejected because: Adds external dependency for functionality already provided by native JavaScript; no significant functional advantage for simple predicate-based queries in test contexts
  When valid: When complex query operations (deep property access, chaining) are frequently needed across many test files
- Pre-index collections into Map structures keyed by identifier for O(1) lookup (deferred)
  Rejected because: Adds setup complexity and memory overhead for test data structures; current evidence shows small collection sizes where linear search is acceptable
  When valid: When test collections grow large enough that linear search becomes measurable performance bottleneck

## Risks

- Linear search performance degrades with large collections, potentially slowing test execution
  Mitigation: Monitor test execution times; consider indexing strategy if collections exceed 100 items or tests show measurable slowdown
  Owner: engineering team
- Undefined results from Array.find() may cause test failures if not properly checked before property access
  Mitigation: Enforce expect().toBeDefined() assertions immediately after find() operations; use TypeScript strict null checks in test files
  Owner: engineering team
- Complex multi-condition predicates reduce test readability and increase maintenance burden
  Mitigation: Extract complex predicate logic into named helper functions; limit predicate complexity to 2-3 conditions maximum
  Owner: engineering team

## Implementation Notes

- Always verify the result of Array.find() with expect().toBeDefined() before accessing properties in assertions
- Use arrow functions for simple single-condition predicates; extract named functions for complex multi-condition logic
- Store found entities in descriptive variable names when multiple assertions are performed on the same entity
- Consider Array.filter() instead of find() when test needs to assert on multiple matching items or collection size

## Continuation Context


Verify commands:
- grep -r 'Array\.find\|allAgents\.find\|\.find(' tests/ --include='*.test.ts' | wc -l
- grep -r '\.find(.*=>.*getIdentifier\|code ===' tests/ --include='*.test.ts'
- npm test -- --testPathPattern='integration|unified-config' --passWithNoTests

Accept when:
- Grep commands identify at least 2 instances of .find() usage with predicate functions in test files
- Test files contain expect().toBeDefined() or similar checks after find() operations before property access
- Integration tests execute successfully without undefined reference errors from collection queries

## Enforcement

- Verified by: Code review of test files checking for Array.find() usage patterns
- Verified by: Static analysis or linting rules detecting .find() without subsequent undefined checks
- Verified by: Test execution in CI pipeline validating no runtime errors from collection queries
- Violation handling: Code review feedback requesting refactor to Array.find() pattern for single-item collection queries
- Violation handling: Test failures from undefined property access trigger investigation of find() result handling
- Violation handling: Pull request comments highlighting missing toBeDefined() assertions after find() operations
- Exception process: Document rationale in test file comments when alternative query pattern is required
- Exception process: Obtain approval from test infrastructure maintainers for deviations in performance-critical scenarios
- Exception process: Record exception in test suite documentation with justification and alternative approach used