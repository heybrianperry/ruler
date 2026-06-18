# Standardize Array.find() for Safe Data Access in Test Assertions: Predicate Functions Passed

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Test suites for MistralVibeAgent and apply-engine require safe access to parsed configuration data structures, including TOML-parsed MCP server lists and hierarchical ruler configurations
- Direct array indexing or unsafe property access in test assertions can produce false negatives when expected data is missing or malformed, masking configuration parsing failures
- The codebase uses Array.find() with predicate functions to locate specific configuration objects by name or path properties, enabling explicit null-handling in assertions
- Tests validate both positive cases (configuration present) and negative cases (configuration absent or malformed), requiring a pattern that distinguishes between undefined results and parsing errors

## Problem Statement

Test code that accesses parsed configuration data without explicit existence checks can silently fail or produce misleading test results when configurations are missing, malformed, or incorrectly parsed. This undermines test reliability and obscures actual configuration validation failures, particularly in security-sensitive contexts like MCP server configuration and hierarchical rule loading.

## Decision

1. SHOULD: Predicate functions passed to Array.find() SHOULD match on stable identifier properties (name, id, path) rather than positional indices

## Policy Block

- SHOULD Predicate functions passed to Array.find() SHOULD match on stable identifier properties (name, id, path) rather than positional indices

In scope:
- Test files accessing parsed TOML configuration data (MCP servers, agent configurations)
- Test files accessing hierarchical configuration structures (ruler directories, nested modules)
- Test assertions that validate presence or absence of specific configuration entries
- Test code that validates JSON.parse() or parseTOML() output structures

Out of scope:
- Production code data access patterns (covered by separate ADRs)
- Test code accessing in-memory data structures created within the test itself
- Simple array iteration where all elements are processed uniformly
- Test fixtures with guaranteed structure and known indices

Exceptions:
- EXC-001: Test explicitly validates array indexing behavior or boundary conditions
- EXC-002: Test uses array destructuring with default values that provide safe fallbacks

## Rationale

- Evidence shows consistent use of Array.find() with name-based predicates across MistralVibeAgent.test.ts (4 instances matching MCP server names) and apply-engine.test.ts (4 instances matching ruler directory paths), indicating an established pattern for safe configuration access
- The pattern enables test assertions to distinguish between 'configuration not found' (undefined) and 'configuration malformed' (present but invalid), improving diagnostic clarity when tests fail
- Using predicate-based search rather than positional indexing makes tests resilient to configuration ordering changes and more clearly expresses test intent
- The pattern aligns with secure coding practices by preventing undefined behavior from unsafe property access on potentially missing objects

## Consequences

Positive:
- Test failures provide clearer diagnostic information by distinguishing missing configurations from malformed ones
- Tests become resilient to configuration ordering changes, reducing brittleness
- Explicit undefined checks prevent cascading test failures from unsafe property access
- Pattern encourages test authors to consider both positive and negative cases for configuration presence

Negative:
- Slightly more verbose test code compared to direct array indexing
- Requires test authors to understand predicate function syntax and undefined handling
- May introduce performance overhead in tests with large configuration arrays (typically negligible in test contexts)

## Alternatives

- Use direct array indexing with length checks (e.g., if (array.length > 0) array[0]) (rejected)
  Rejected because: Positional indexing couples tests to configuration ordering and does not express semantic intent (finding by name/path). Length checks do not guarantee the expected item is at the checked index.
  When valid: When configuration order is guaranteed and semantically meaningful, such as testing array sorting behavior
- Use optional chaining (array[0]?.property) without explicit find (rejected)
  Rejected because: Optional chaining silences errors but does not solve the positional indexing problem. Tests may pass when they should fail if the expected configuration is present but at a different index.
  When valid: For accessing deeply nested optional properties after a safe find operation
- Create custom test utility functions (e.g., findConfigByName) that wrap Array.find() (deferred)
  Rejected because: Not rejected; marked as optional (MAY) in R-42-006. Could be adopted if the pattern becomes more widespread across the test suite.
  When valid: When multiple test files show repetitive Array.find() patterns with identical predicate logic

## Risks

- Existing test code using direct indexing may not be identified and updated, creating inconsistent patterns across the test suite
  Mitigation: Run grep/ripgrep searches for array indexing patterns in test files; add linting rule to flag direct indexing on parsed configuration objects
  Owner: Engineering team
- Test authors unfamiliar with Array.find() may write incorrect predicates that always return undefined
  Mitigation: Provide test code examples in documentation; include negative test cases that verify find() returns undefined for non-existent items
  Owner: Engineering team
- Performance degradation in tests with very large configuration arrays requiring multiple find() operations
  Mitigation: Profile test execution times; consider caching find() results in test setup if multiple assertions access the same configuration object
  Owner: Engineering team

## Implementation Notes

- When writing tests for configuration parsing, structure assertions as: const found = array.find(predicate); expect(found).toBeDefined(); expect(found?.property).toBe(expectedValue);
- Use descriptive predicate variable names that clarify the search intent, e.g., (s: any) => s.name === 'expected_server' rather than (s) => s.name === 'expected_server'
- For tests validating multiple configuration objects, consider extracting find() calls to test setup (beforeEach) with clear variable names to reduce repetition
- Include at least one negative test case per configuration type that verifies find() returns undefined for non-existent entries

## Continuation Context


Verify commands:
- grep -r 'parsed.*\[0\]' tests/ --include='*.test.ts' | grep -v 'length' # Flag direct indexing without length checks
- grep -r '\.find(' tests/ --include='*.test.ts' | wc -l # Count Array.find() usage in tests
- npm test -- --testPathPattern='MistralVibeAgent|apply-engine' # Verify reference tests pass

Accept when:
- All test files accessing parsed configuration arrays use Array.find() with explicit predicates
- Test assertions include explicit undefined checks before accessing properties on find() results
- Grep search for direct array indexing on parsed configuration objects returns zero results in test files

## Enforcement

- Verified by: Code review checklist item for test files accessing parsed configurations
- Verified by: Automated grep/ripgrep checks in CI pipeline flagging direct array indexing patterns
- Verified by: Test coverage reports ensuring negative test cases exist for configuration access
- Violation handling: CI pipeline warnings for direct array indexing in test files
- Violation handling: Code review feedback requesting refactor to Array.find() pattern
- Violation handling: Test failures due to unsafe property access trigger immediate remediation
- Exception process: Test author documents exception rationale in test description or comment
- Exception process: Code reviewer approves exception based on documented justification
- Exception process: Exception is recorded in test file with EXC-ID reference for traceability