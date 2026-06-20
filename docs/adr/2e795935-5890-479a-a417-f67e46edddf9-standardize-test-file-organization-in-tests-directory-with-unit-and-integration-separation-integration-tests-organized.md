# Standardize Test File Organization in tests/ Directory with Unit and Integration Separation: Integration Tests Organized

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all test file creation and organization within the codebase.

## Context

- The codebase contains a consistent pattern of test organization with 9 test files following a structured approach separating unit tests from integration tests
- Test files are organized under a tests/ directory with clear subdirectories for tests/unit/ and tests/integration/, enabling different test execution strategies and CI/CD pipeline optimization
- The pattern shows testing across multiple layers including core engine components (revert-engine, apply-engine, RuleProcessor), agent implementations (FirebenderAgent), and integration scenarios (skills-mcp-agent-integration, hierarchical-rules)
- The testing.mocking facet indicates the use of mocking strategies within the test suite, suggesting a mature testing approach that isolates dependencies
- This organizational pattern supports CI/CD workflows by allowing selective test execution based on test type (unit vs integration) and enables faster feedback loops during development

## Problem Statement

Without a standardized test organization structure, test files become scattered across the codebase, making it difficult to execute specific test suites, optimize CI/CD pipelines, and maintain clear boundaries between fast unit tests and slower integration tests. This leads to longer build times, unclear test coverage, and difficulty in implementing effective testing strategies.

## Decision

1. MUST: Integration tests MUST be organized under tests/integration/ subdirectory

## Policy Block

- MUST Integration tests MUST be organized under tests/integration/ subdirectory

In scope:
- All TypeScript and JavaScript test files
- Unit tests for core engine components, agents, and utilities
- Integration tests for multi-component interactions
- Test configuration files within the tests/ directory

Out of scope:
- Test fixtures and mock data files (may be organized separately)
- Test utilities and helper functions (may be in tests/helpers/ or tests/utils/)
- Documentation and README files within test directories
- Temporary test output files

## Rationale

- The detected pattern shows 9 files with 92.37% confidence following this organizational structure, indicating a well-established convention in the codebase
- Separating unit and integration tests enables CI/CD pipelines to run fast unit tests on every commit while reserving slower integration tests for specific stages or branches
- Mirroring source code structure in unit tests makes it intuitive to locate tests for specific modules and ensures comprehensive coverage tracking
- The testing.mocking facet indicates mature testing practices that benefit from clear organizational boundaries between test types

## Consequences

Positive:
- Faster CI/CD feedback loops by enabling selective execution of unit tests vs integration tests
- Improved developer experience with predictable test file locations following source code structure
- Better test coverage visibility and reporting by test type
- Easier onboarding for new developers with clear, consistent test organization
- Optimized test execution strategies in CI pipelines (parallel execution, caching, etc.)

Negative:
- Requires discipline to maintain the organizational structure as the codebase grows
- May require refactoring existing tests that don't follow the convention
- Additional directory depth may make file paths longer
- Edge cases (e.g., tests that span unit/integration boundaries) require judgment calls on placement

## Alternatives

- Co-locate tests with source files (e.g., src/core/RuleProcessor.test.ts alongside RuleProcessor.ts) (rejected)
  Rejected because: Co-location makes it harder to execute test suites selectively, complicates build configurations, and mixes production and test code in the same directories
  When valid: May be appropriate for very small projects or specific frameworks that strongly encourage co-location
- Flat test directory structure without unit/integration separation (all tests in tests/) (rejected)
  Rejected because: Flat structure prevents selective test execution in CI/CD pipelines and makes it difficult to distinguish between fast unit tests and slower integration tests
  When valid: Only suitable for very small projects with fewer than 10 test files
- Organize tests by feature rather than by test type (e.g., tests/revert/, tests/apply/) (rejected)
  Rejected because: Feature-based organization doesn't support CI/CD optimization strategies that depend on test type separation and makes it harder to run all unit tests quickly
  When valid: Could be used as a secondary organizational layer within unit/ or integration/ subdirectories for very large codebases

## Risks

- Developers may inconsistently apply the organizational structure, leading to test files in wrong locations
  Mitigation: Implement automated linting rules and CI checks to verify test file locations match the required structure
  Owner: Engineering team
- Ambiguity in classifying tests as unit vs integration may lead to misplacement
  Mitigation: Document clear criteria for unit vs integration classification and provide examples in testing guidelines
  Owner: Engineering team
- Legacy tests may not follow the structure, creating inconsistency during transition period
  Mitigation: Create a migration plan to gradually refactor existing tests and track progress with metrics
  Owner: Engineering team

## Implementation Notes

- Configure test runners (Jest, Mocha, etc.) to recognize the tests/ directory structure and support running unit and integration tests separately
- Update CI/CD pipeline configurations to execute unit tests on every commit and integration tests on pull requests or specific branches
- Create test templates or scaffolding tools that automatically place new test files in the correct location based on the source file being tested
- Document the organizational structure in the project's testing guidelines with clear examples of unit vs integration test placement
- Consider using path aliases or test utilities to simplify imports from deeply nested test files

## Continuation Context


Verify commands:
- find tests/ -name '*.test.ts' -o -name '*.test.js' | grep -v -E '^tests/(unit|integration)/' && echo 'Found tests outside unit/integration dirs' || echo 'All tests properly organized'
- test -d tests/unit && test -d tests/integration && echo 'Required test directories exist' || echo 'Missing required test directories'
- npm test -- --testPathPattern=tests/unit --listTests | wc -l

Accept when:
- All test files with .test.ts or .test.js extension are located under tests/unit/ or tests/integration/ directories
- Test runner can successfully execute unit tests and integration tests as separate suites
- CI/CD pipeline successfully runs unit tests independently from integration tests with appropriate timing thresholds

## Enforcement

- Verified by: Automated CI checks that scan for test files outside the approved directory structure
- Verified by: Code review checklist items verifying test file placement
- Verified by: Pre-commit hooks that validate test file locations
- Violation handling: CI pipeline fails if test files are found outside tests/unit/ or tests/integration/ directories
- Violation handling: Pull requests with incorrectly placed tests are flagged for revision during code review
- Violation handling: Automated comments on PRs suggest correct test file locations when violations are detected
- Exception process: Exceptions for specialized test types (e.g., e2e, performance) must be documented in testing guidelines
- Exception process: New test directory types require approval from tech lead and must follow the same organizational principles
- Exception process: Temporary exceptions during migration periods must be tracked in a technical debt register with remediation timeline