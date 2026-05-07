# Adopt TypeScript-based Unit Testing Framework for Agent and CLI Components: Test Files Use

Status: proposed
Date: 2025-01-10
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all TypeScript test development in the codebase.

## Context

- The codebase contains multiple agent implementations (WarpAgent, JulesAgent, JunieAgent, OpenCodeAgent, WindsurfAgent, AmpAgent) that require consistent testing approaches to ensure reliability and maintainability.
- A standardized unit testing framework pattern has emerged across 8 test files with 92.19% confidence, indicating a deliberate architectural choice for test organization.
- The test files follow a consistent naming convention (*.test.ts) and directory structure (tests/unit/) suggesting an established testing culture and tooling setup.
- TypeScript is used as the primary language for both implementation and testing, enabling type-safe test development and better IDE support.
- The pattern includes both component-level tests (agents) and infrastructure tests (CLI handlers), indicating comprehensive test coverage requirements.

## Problem Statement

Without a standardized testing framework and consistent test organization patterns, the codebase risks inconsistent test quality, duplicated test utilities, difficulty in maintaining tests across multiple agent implementations, and reduced developer productivity when writing or debugging tests. A unified approach is needed to ensure all components follow the same testing conventions and leverage shared testing infrastructure.

## Decision

1. SHOULD: Test files SHOULD use a consistent testing framework (e.g., Jest, Vitest, or Mocha) with TypeScript support across all unit tests.

## Policy Block

- SHOULD Test files SHOULD use a consistent testing framework (e.g., Jest, Vitest, or Mocha) with TypeScript support across all unit tests.

In scope:
- All TypeScript unit tests for agent implementations
- All TypeScript unit tests for CLI handlers and commands
- Test files for core business logic and component behavior
- Error handling and validation test cases

Out of scope:
- Integration tests that span multiple components or external services
- End-to-end tests that test the full application workflow
- Performance or load testing suites
- Tests written in languages other than TypeScript (if any exist)
- Manual testing procedures or QA scripts

Exceptions:
- EXC-001: Legacy JavaScript test files that predate the TypeScript migration
- EXC-002: Third-party or vendor-specific testing tools that require alternative file structures

## Rationale

- The pattern detection identified 8 files with 92.19% confidence, demonstrating strong consistency in the testing approach across the codebase.
- TypeScript provides type safety for tests, catching errors earlier and improving test reliability through compile-time checks.
- Consistent test organization (tests/unit/ structure with mirrored source paths) makes tests discoverable and maintainable as the codebase grows.
- The presence of tests for multiple agent types (7 different agents) indicates a scalable pattern that supports the plugin/agent architecture of the system.

## Consequences

Positive:
- Developers can quickly locate and understand tests due to consistent naming and organization conventions.
- TypeScript's type system catches test errors during development, reducing test flakiness and debugging time.
- New agent implementations can follow established testing patterns, accelerating development velocity.
- Automated tooling (linters, test runners, CI/CD) can reliably discover and execute tests based on consistent patterns.
- Shared testing infrastructure and utilities can be developed once and reused across all test files.

Negative:
- Requires TypeScript compilation step for tests, adding slight overhead to test execution time.
- Developers must maintain TypeScript knowledge for both implementation and testing.
- Strict adherence to directory structure may require refactoring if source code organization changes.
- May create initial friction for developers accustomed to different testing frameworks or patterns.

## Alternatives

- Use JavaScript (.test.js) for tests instead of TypeScript (rejected)
  Rejected because: JavaScript tests would lose type safety benefits and create inconsistency with the TypeScript codebase, leading to runtime errors that could be caught at compile time.
  When valid: Only valid for projects that are entirely JavaScript-based without TypeScript adoption plans.
- Co-locate tests with source files (e.g., src/agents/WarpAgent.test.ts) (rejected)
  Rejected because: The detected pattern shows clear separation with tests/unit/ structure, which provides cleaner separation of concerns and easier exclusion from production builds.
  When valid: Valid for smaller projects or when test proximity to source is prioritized over separation.
- Use a single test file per module with all tests combined (rejected)
  Rejected because: The pattern shows one test file per agent/component, providing better test isolation and maintainability as components grow in complexity.
  When valid: Valid only for very simple modules with minimal test cases.

## Risks

- Test framework lock-in: Heavy investment in a specific TypeScript testing framework could make migration difficult if the framework becomes unmaintained or inadequate.
  Mitigation: Abstract common testing patterns behind utility functions and maintain framework-agnostic test structure where possible. Monitor framework health and community support.
  Owner: Engineering Team / Test Infrastructure Lead
- Test organization drift: As the codebase grows, developers may deviate from the established patterns without enforcement mechanisms.
  Mitigation: Implement automated checks in CI/CD to verify test file naming and location conventions. Include test organization guidelines in developer onboarding documentation.
  Owner: DevOps Team / Engineering Leads
- TypeScript compilation overhead may slow down test feedback loops, especially as test suite grows.
  Mitigation: Use incremental TypeScript compilation, implement test file watching with fast refresh, and consider test parallelization strategies.
  Owner: Engineering Team / Performance Lead

## Implementation Notes

- Ensure tsconfig.json includes test directories and has appropriate compiler options for test execution (e.g., esModuleInterop, allowSyntheticDefaultImports).
- Configure the chosen test framework (Jest/Vitest) with TypeScript preset or ts-jest transformer to handle .test.ts files.
- Create a test template or generator script to scaffold new test files following the established pattern (e.g., npm run generate:test).
- Document the testing conventions in a TESTING.md file at the repository root, including examples from existing tests.
- Set up IDE configurations (VSCode settings, WebStorm run configurations) to recognize and execute .test.ts files with proper TypeScript support.
- Consider creating shared test utilities in tests/utils/ or tests/helpers/ for common mocking, fixtures, and assertion helpers.

## Continuation Context


Verify commands:
- find tests/unit -name '*.test.ts' -type f | wc -l
- grep -r '\.test\.js' tests/unit/ || echo 'No JavaScript test files found'
- npm test -- --listTests | grep -E 'tests/unit/.*\.test\.ts$'
- npx tsc --noEmit --project tsconfig.test.json

Accept when:
- All unit test files in tests/unit/ directory use .test.ts extension and compile successfully with TypeScript.
- Test directory structure mirrors source code organization with agents tests in tests/unit/agents/ and CLI tests in tests/unit/cli/.
- No JavaScript (.test.js) unit test files exist in the tests/unit/ directory structure.
- TypeScript compilation of test files produces no type errors when running tsc --noEmit.

## Enforcement

- Verified by: CI/CD pipeline runs automated checks to verify test file naming conventions and locations.
- Verified by: TypeScript compiler enforces type correctness for all test files during pre-commit hooks.
- Verified by: Code review checklist includes verification that new components have corresponding .test.ts files in correct locations.
- Verified by: Automated test discovery in CI fails if tests are not in expected locations or use incorrect extensions.
- Violation handling: CI/CD pipeline fails if test files are found outside the tests/unit/ structure or use incorrect extensions.
- Violation handling: Pull requests with missing unit tests for new agents or CLI handlers are flagged for review.
- Violation handling: Linting rules flag test files that don't follow naming conventions.
- Violation handling: Quarterly audits identify and remediate any test organization drift.
- Exception process: Developer submits exception request via GitHub issue or architecture review board with justification.
- Exception process: Tech Lead or Architecture Review Board evaluates exception against policy scope and alternatives.
- Exception process: Approved exceptions are documented in test file headers and tracked in exceptions registry.
- Exception process: All exceptions include remediation plan or sunset date for temporary deviations.