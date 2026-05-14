# Adopt TypeScript with Jest/Vitest for Unit Testing Framework: Test Files Written

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all TypeScript test file development and test framework configuration.

## Context

- The codebase contains 8 TypeScript test files following a consistent .test.ts naming convention, indicating a standardized testing approach across multiple agent implementations (AmpAgent, WarpAgent, JulesAgent, WindsurfAgent, OpenCodeAgent, JunieAgent) and CLI handlers.
- Test files are organized in a structured directory hierarchy under tests/unit/, with agent-specific tests grouped in tests/unit/agents/ and CLI tests in tests/unit/cli/, demonstrating a clear separation of concerns.
- The pattern shows 92.19% confidence across all detected files, suggesting this is an established and consistently applied testing standard rather than an ad-hoc approach.
- The presence of an invalid-agent.test.ts file indicates the testing framework is used for both positive and negative test scenarios, supporting comprehensive test coverage.
- TypeScript test files enable type-safe test development, catching type errors at compile time and providing better IDE support for test authoring.

## Problem Statement

Without a standardized testing framework and file organization convention, test code becomes inconsistent across the codebase, making it difficult to maintain test suites, onboard new developers, and ensure comprehensive coverage. The lack of type safety in tests can lead to runtime errors that could have been caught during development.

## Decision

1. MUST: Test files MUST be written in TypeScript to leverage type safety and maintain consistency with the application codebase.

## Policy Block

- MUST Test files MUST be written in TypeScript to leverage type safety and maintain consistency with the application codebase.

In scope:
- All TypeScript unit test files in the codebase
- Test files for agent implementations, CLI handlers, and core business logic
- Automated test discovery and execution pipelines
- Developer tooling configuration (test runners, IDE plugins, linters)

Out of scope:
- Integration tests or end-to-end tests (which may use different conventions)
- Performance or load testing files
- Test fixtures, mocks, or helper utilities (which may use different naming patterns)
- Documentation or example code that is not part of the automated test suite

Exceptions:
- EX-001: Legacy JavaScript test files exist and are being incrementally migrated to TypeScript
- EX-002: Specialized testing tools require alternative file extensions (e.g., .spec.ts for specific frameworks)

## Rationale

- The consistent .test.ts naming convention across 8 files with 92.19% confidence indicates this is an established pattern that has proven effective for the team's workflow and tooling integration.
- TypeScript test files provide compile-time type checking, reducing runtime errors in tests and improving developer productivity through better IDE autocomplete and refactoring support.
- Organizing tests in a parallel directory structure (tests/unit/) keeps test code separate from production code while maintaining clear correspondence between source and test files.
- The pattern supports both positive test cases (agent implementations) and negative test cases (invalid-agent.test.ts), enabling comprehensive test coverage strategies.

## Consequences

Positive:
- Consistent test file naming enables automatic test discovery by popular test runners (Jest, Vitest, ts-node), reducing configuration overhead.
- Type safety in tests catches errors earlier in the development cycle, improving code quality and reducing debugging time.
- Clear directory organization makes it easy for developers to locate tests for specific modules and understand the test coverage landscape.
- Standardized conventions reduce cognitive load during code reviews and make it easier to onboard new team members to the testing practices.

Negative:
- Requires TypeScript compilation step for tests, potentially increasing test execution time compared to plain JavaScript tests.
- Developers must maintain TypeScript type definitions for test utilities and mocks, adding some overhead to test development.
- Migration of existing JavaScript tests to TypeScript requires effort and may temporarily create inconsistency during transition periods.
- Strict adherence to directory structure may require refactoring if source code organization changes significantly.

## Alternatives

- Use .spec.ts extension instead of .test.ts for test files (rejected)
  Rejected because: The detected pattern shows consistent use of .test.ts across all 8 files, and changing would break existing test discovery configuration and team conventions.
  When valid: Valid for new projects where no existing convention exists, or if migrating to a framework that strongly prefers .spec.ts (e.g., Angular)
- Co-locate test files with source code using __tests__/ subdirectories (rejected)
  Rejected because: The evidence shows tests are organized in a separate tests/unit/ hierarchy, which provides clearer separation between production and test code.
  When valid: Valid for smaller modules or libraries where keeping tests close to source code improves discoverability and reduces navigation overhead
- Use plain JavaScript (.test.js) for tests to avoid TypeScript compilation overhead (rejected)
  Rejected because: All detected test files use TypeScript (.test.ts), indicating the team values type safety in tests despite the compilation overhead.
  When valid: Valid for projects with minimal type complexity or where test execution speed is critical and type safety is less important

## Risks

- Test execution time may increase due to TypeScript compilation, potentially slowing down development feedback loops.
  Mitigation: Use incremental compilation, watch mode, and caching strategies in the test runner. Consider using ts-jest or Vitest with optimized TypeScript handling.
  Owner: Engineering Team / DevOps
- Inconsistent application of the .test.ts convention could lead to tests being missed by the test runner, creating false confidence in test coverage.
  Mitigation: Implement pre-commit hooks and CI checks to verify all test files follow the naming convention. Add linting rules to enforce test file patterns.
  Owner: Engineering Team / QA
- Developers unfamiliar with TypeScript may face a steeper learning curve when writing tests, potentially reducing test coverage or quality.
  Mitigation: Provide TypeScript testing documentation, examples, and templates. Conduct training sessions and pair programming for knowledge transfer.
  Owner: Engineering Team / Tech Lead

## Implementation Notes

- Configure the test runner (Jest or Vitest) to automatically discover files matching the **/*.test.ts pattern in the tests/ directory.
- Set up TypeScript compilation for tests with appropriate tsconfig.json settings, ensuring test files have access to necessary type definitions and can import from source code.
- Create test file templates or code snippets in the IDE to help developers quickly scaffold new test files following the established conventions.
- Document the testing conventions in the project README or contributing guidelines, including examples of proper test file organization and naming.
- Implement CI/CD pipeline checks to ensure all .test.ts files are discovered and executed, and that test coverage meets minimum thresholds.

## Continuation Context


Verify commands:
- find tests/unit -name '*.test.ts' -type f | wc -l
- grep -r "describe\|test\|it" tests/unit/**/*.test.ts | head -n 5
- npx tsc --noEmit --project tsconfig.test.json 2>&1 | grep -c 'error' || echo '0'

Accept when:
- All test files in tests/unit/ directory use the .test.ts extension and are discoverable by the test runner
- TypeScript compilation succeeds for all test files without type errors
- Test runner configuration correctly identifies and executes all .test.ts files, with no orphaned or undiscovered tests

## Enforcement

- Verified by: Automated CI/CD pipeline checks that verify test file naming conventions
- Verified by: Pre-commit hooks that validate test file extensions and directory structure
- Verified by: Code review process that checks for proper test file organization and TypeScript usage
- Verified by: Periodic audits of test coverage reports to identify missing or misconfigured test files
- Violation handling: CI build fails if test files do not follow the .test.ts naming convention or are not in the correct directory structure
- Violation handling: Pull requests are blocked until test files are properly named and organized according to the standard
- Violation handling: Automated notifications are sent to developers when violations are detected, with guidance on how to fix them
- Violation handling: Test coverage reports flag files without corresponding .test.ts files for review
- Exception process: Developer submits exception request to Tech Lead with justification for deviation from standard (e.g., framework-specific requirements, legacy migration)
- Exception process: Tech Lead reviews request and approves/rejects based on technical merit and impact on codebase consistency
- Exception process: Approved exceptions are documented in the project's ADR addendum or testing guidelines with clear scope and expiration date
- Exception process: Exceptions are reviewed quarterly to determine if they can be eliminated or if the standard should be updated