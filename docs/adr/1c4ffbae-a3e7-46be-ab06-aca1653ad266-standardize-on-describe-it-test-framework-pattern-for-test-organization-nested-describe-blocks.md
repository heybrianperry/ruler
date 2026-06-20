# Standardize on describe/it Test Framework Pattern for Test Organization: Nested Describe Blocks

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all test files in the codebase and applies to all test suite implementations.

## Context

- The codebase contains 37 test files exhibiting a consistent pattern of using describe/it test framework constructs for organizing test suites and test cases
- Test files span multiple domains including agent implementations, CLI behavior, configuration loading, MCP integration, and skills processing, indicating organization-wide adoption
- Evidence shows systematic use of beforeEach/afterEach lifecycle hooks for test setup and teardown across integration and unit tests
- Test organization follows a hierarchical structure with describe blocks for grouping related tests and it blocks for individual test cases
- The pattern appears in both unit tests (tests/unit/agents/) and integration tests (tests/integration/), suggesting a unified testing approach across test types

## Problem Statement

Without a standardized test organization framework, test suites become inconsistent in structure, making them harder to navigate, maintain, and extend. The codebase requires a consistent approach to organizing test cases, managing test lifecycle, and expressing test intent across unit, integration, and functional test boundaries.

## Decision

1. SHOULD: Nested describe blocks SHOULD be used to organize tests hierarchically when testing multiple aspects of a component or feature

## Policy Block

- SHOULD Nested describe blocks SHOULD be used to organize tests hierarchically when testing multiple aspects of a component or feature

In scope:
- All test files in tests/ directory including unit, integration, and functional tests
- Test files for agent implementations (tests/unit/agents/)
- Test files for CLI behavior and configuration loading
- Test files for MCP integration and skills processing

Out of scope:
- Non-test source code files
- Build scripts and tooling configuration
- Documentation and example code that is not executed as part of the test suite

## Rationale

- The describe/it pattern provides a clear hierarchical structure that maps naturally to the component-feature-behavior organization needed for comprehensive test coverage
- Consistent test organization across 37 files with 92.59% confidence indicates this pattern has proven effective for the codebase's testing needs
- Lifecycle hooks (beforeEach/afterEach) enable proper test isolation by ensuring each test starts with clean state, preventing test interdependencies
- The pattern is framework-agnostic in concept but typically implemented via Jest, Mocha, or similar BDD-style test frameworks, providing portability and familiarity for developers

## Consequences

Positive:
- Test suites have consistent structure making them easier to navigate and understand for new contributors
- Hierarchical organization with describe blocks enables logical grouping of related tests and better test output reporting
- Lifecycle hooks ensure proper test isolation and reduce boilerplate setup code duplication
- Natural language test descriptions serve as living documentation of system behavior and requirements

Negative:
- Teams must learn and follow the describe/it convention, which may require training for developers unfamiliar with BDD-style testing
- Deeply nested describe blocks can become difficult to follow if not managed carefully
- Lifecycle hooks can hide setup complexity and make individual tests harder to understand in isolation
- The pattern requires a compatible test framework (Jest, Mocha, Jasmine) which constrains testing tool choices

## Alternatives

- Use flat test functions without describe/it grouping (e.g., test_feature_behavior naming convention) (rejected)
  Rejected because: Flat structure lacks hierarchical organization and makes it difficult to group related tests or generate meaningful test reports with nested context
  When valid: May be appropriate for very simple test suites with fewer than 10 test cases and no need for shared setup
- Use class-based test organization with setUp/tearDown methods (xUnit style) (rejected)
  Rejected because: Class-based approach is more verbose and less idiomatic in JavaScript/TypeScript ecosystems where the codebase operates
  When valid: Valid for codebases using languages with strong OOP conventions like Java or C# where xUnit patterns are standard
- Mix multiple test organization patterns based on developer preference (rejected)
  Rejected because: Inconsistent patterns across the codebase increase cognitive load and make test maintenance more difficult
  When valid: Never valid for a cohesive codebase; consistency is critical for maintainability

## Risks

- Developers may create overly complex nested describe structures that obscure test intent
  Mitigation: Establish code review guidelines limiting describe nesting to 3 levels maximum and requiring clear naming at each level
  Owner: Engineering team
- Lifecycle hooks may introduce hidden dependencies between tests if not properly isolated
  Mitigation: Enforce that each test can run independently and use linting rules to detect shared mutable state in hooks
  Owner: Engineering team
- Test framework lock-in may complicate future migrations to alternative testing tools
  Mitigation: Document the describe/it pattern as a conceptual standard and maintain awareness that migration would require systematic refactoring
  Owner: Engineering team

## Implementation Notes

- Use describe blocks to name the component or feature under test, and nest additional describe blocks for specific aspects or scenarios
- Write it block descriptions as complete sentences starting with 'should' to clearly express expected behavior (e.g., 'should return error when input is invalid')
- Place common setup logic in beforeEach hooks and cleanup logic in afterEach hooks to ensure test isolation
- Keep test files focused on a single component or feature area to maintain clear test organization and avoid overly large test suites

## Continuation Context


Verify commands:
- grep -r "describe(" tests/ | wc -l
- grep -r "it(" tests/ | wc -l
- grep -r "beforeEach\|afterEach" tests/ | wc -l

Accept when:
- All test files in tests/ directory use describe blocks for test suite organization
- Individual test cases are defined using it blocks with descriptive behavior-focused names
- Test files requiring setup/teardown use beforeEach/afterEach hooks appropriately

## Enforcement

- Verified by: Code review process checks for consistent use of describe/it pattern in new test files
- Verified by: Linting rules detect test files that do not follow the describe/it structure
- Verified by: CI pipeline runs test suite and verifies all tests are properly organized and executable
- Violation handling: Pull requests with non-compliant test organization are flagged during code review
- Violation handling: Developers are asked to refactor tests to use describe/it pattern before merge
- Violation handling: Existing non-compliant tests are tracked as technical debt for gradual refactoring
- Exception process: Exceptions require explicit justification in code comments explaining why the standard pattern cannot be used
- Exception process: Exception requests are reviewed by tech lead or senior engineer
- Exception process: Approved exceptions are documented in test file headers with rationale and expected duration