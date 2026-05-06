# Standardize Unit and Integration Test Organization in tests/ Directory: Test Files Not

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all test file organization and CI/CD pipeline configuration.

## Context

- The codebase contains 84 test files following a consistent organizational pattern with tests/ as the root directory, indicating a mature testing infrastructure
- Test files are segregated into tests/unit/ and tests/integration/ subdirectories, suggesting a deliberate separation of test types for different execution contexts
- The pattern shows tests organized by component type (core/, agents/) and by test scope (unit vs integration), enabling selective test execution in CI/CD pipelines
- Test file naming follows the convention *.test.ts, providing clear identification of test artifacts and enabling glob-based test discovery
- The high confidence (91.89%) and support count (84 files) indicate this is an established, organization-wide standard rather than an isolated practice

## Problem Statement

Without a standardized test organization structure, teams face challenges in maintaining consistent CI/CD pipelines, executing targeted test suites, and managing test execution time. Ad-hoc test placement leads to difficulty in distinguishing unit tests from integration tests, prevents parallel test execution strategies, and complicates test coverage reporting and failure analysis.

## Decision

1. MUST_NOT: Test files MUST NOT be co-located with source code in src/ or lib/ directories

## Policy Block

- MUST_NOT Test files MUST NOT be co-located with source code in src/ or lib/ directories

In scope:
- All TypeScript/JavaScript test files in the project
- Unit tests that test individual functions, classes, or modules in isolation
- Integration tests that test interactions between multiple components or external systems
- CI/CD pipeline test execution configuration
- Test coverage reporting and analysis tools

Out of scope:
- End-to-end (E2E) tests which may warrant a separate tests/e2e/ directory
- Performance or load tests which may use different tooling
- Manual test plans or documentation
- Test fixtures or mock data organization (may follow different conventions)
- Language-specific test frameworks for non-TypeScript code

Exceptions:
- EXC-001: Legacy test files exist in non-standard locations during migration period
- EXC-002: Framework-specific tests require co-location (e.g., Storybook stories with .test suffix)

## Rationale

- The pattern is detected across 84 files with 91.89% confidence, demonstrating strong organizational consensus and proven effectiveness
- Separating unit and integration tests enables differential CI/CD strategies: fast unit tests on every commit, slower integration tests on merge or nightly builds
- Consistent test organization reduces cognitive load for developers navigating the codebase and makes test discovery predictable
- Standardized structure enables tooling automation for test execution, coverage reporting, and failure analysis without custom configuration per project

## Consequences

Positive:
- CI/CD pipelines can selectively execute test suites (unit vs integration) based on directory structure, optimizing build times
- Developers can quickly locate and run relevant tests without searching through source directories
- Test coverage tools can generate separate reports for unit and integration coverage, providing better visibility into test quality
- New team members can immediately understand test organization without consulting documentation
- Parallel test execution becomes straightforward by partitioning based on directory structure

Negative:
- Requires discipline to maintain separation between unit and integration tests, with potential for misclassification
- May require refactoring existing projects with different test organization patterns
- Additional directory depth can make file paths longer and potentially more cumbersome
- Edge cases (e.g., tests that are neither pure unit nor integration) may not fit cleanly into the binary structure

## Alternatives

- Co-locate tests with source code (e.g., src/module.ts and src/module.test.ts in same directory) (rejected)
  Rejected because: Makes it harder to exclude tests from production builds, complicates test-specific tooling configuration, and prevents easy selective execution of test types
  When valid: May be appropriate for small libraries with minimal test infrastructure
- Use __tests__ directories adjacent to source code following Jest convention (rejected)
  Rejected because: Scatters test files throughout the codebase making it harder to get a holistic view of test coverage and prevents clean separation of unit vs integration tests
  When valid: Valid for projects heavily invested in Jest ecosystem where convention is strongly established
- Flat tests/ directory without unit/integration subdirectories (rejected)
  Rejected because: Prevents selective execution of test types in CI/CD pipelines and loses the ability to apply different execution strategies to different test categories
  When valid: Acceptable for very small projects with fewer than 20 test files where categorization overhead exceeds benefits

## Risks

- Developers may incorrectly classify tests, placing integration tests in unit/ directory or vice versa, leading to inappropriate CI/CD execution
  Mitigation: Provide clear guidelines and examples of unit vs integration test characteristics; implement linting rules to detect common anti-patterns (e.g., network calls in unit tests)
  Owner: Engineering team with QA oversight
- Migration of existing projects with different test structures may be time-consuming and introduce temporary inconsistency
  Mitigation: Create automated migration scripts to move and rename test files; allow grace period with both structures during transition; update CI/CD to handle both patterns temporarily
  Owner: DevOps and engineering leads
- Test execution tooling may require configuration updates to discover tests in new locations
  Mitigation: Update test runner configurations (jest.config.js, etc.) to explicitly include tests/ directory patterns; validate test discovery in CI/CD before enforcing standard
  Owner: DevOps team

## Implementation Notes

- Configure test runners (Jest, Vitest, etc.) with testMatch patterns like ['<rootDir>/tests/**/*.test.ts'] to discover all test files
- Set up separate npm scripts for unit and integration tests: 'npm run test:unit' targeting tests/unit/ and 'npm run test:integration' targeting tests/integration/
- Update CI/CD pipeline to run unit tests on every commit/PR and integration tests on merge to main or on a scheduled basis
- Create project scaffolding templates that include tests/unit/ and tests/integration/ directories with example test files
- Document clear criteria for unit vs integration classification: unit tests have no external dependencies (network, filesystem, database), integration tests validate component interactions

## Continuation Context


Verify commands:
- find tests/unit -name '*.test.ts' -type f | wc -l
- find tests/integration -name '*.test.ts' -type f | wc -l
- grep -r 'describe\|it\|test' tests/ --include='*.test.ts' | head -5

Accept when:
- All test files with *.test.ts extension are located under tests/ directory
- Unit tests are organized under tests/unit/ and integration tests under tests/integration/
- Test runner configuration successfully discovers and executes all tests in the tests/ directory structure
- CI/CD pipeline can independently execute unit and integration test suites

## Enforcement

- Verified by: CI/CD pipeline checks that verify test file locations match required patterns
- Verified by: Pre-commit hooks that validate test files are in correct directories
- Verified by: Code review checklist includes verification of test organization
- Verified by: Automated linting rules that flag test files in non-standard locations
- Violation handling: CI/CD build fails if test files are detected outside tests/ directory structure
- Violation handling: Pull requests with incorrectly placed tests are blocked from merge
- Violation handling: Automated comments on PRs guide developers to correct test placement
- Violation handling: Monthly audit reports identify any non-compliant test files for remediation
- Exception process: Submit exception request to architecture review board with justification
- Exception process: Document specific framework or tooling requirement that necessitates deviation
- Exception process: Obtain tech lead approval with time-bound exception period
- Exception process: Add exception to project README and maintain exception registry