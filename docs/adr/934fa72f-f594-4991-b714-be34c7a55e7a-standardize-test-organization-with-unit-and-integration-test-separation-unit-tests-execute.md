# Standardize Test Organization with Unit and Integration Test Separation: Unit Tests Execute

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains a comprehensive test suite with 9 test files covering core functionality, agents, and integration scenarios
- Tests are organized into distinct unit and integration directories, indicating a deliberate separation of test types based on scope and dependencies
- The testing pattern shows consistent use of mocking facilities (facet: testing.mocking) across unit tests to isolate components from external dependencies
- Integration tests validate cross-component behavior and end-to-end workflows without extensive mocking, complementing the unit test coverage
- This dual-layer testing approach supports both rapid feedback during development and comprehensive validation of system behavior

## Problem Statement

Without a standardized approach to organizing and structuring tests, teams risk creating test suites that are difficult to maintain, slow to execute, and unclear in their purpose. Mixed unit and integration tests lead to unpredictable CI/CD pipeline performance, unclear failure diagnostics, and difficulty in running targeted test subsets during development.

## Decision

1. SHOULD: Unit tests SHOULD execute quickly (under 100ms per test) to enable rapid feedback during development

## Policy Block

- SHOULD Unit tests SHOULD execute quickly (under 100ms per test) to enable rapid feedback during development

In scope:
- All automated test files in JavaScript and TypeScript projects
- Test organization within CI/CD pipeline configurations
- Test execution scripts and npm/package.json test commands
- Mock and stub implementations used in unit tests

Out of scope:
- Manual testing procedures and QA workflows
- Test data generation and fixture management strategies
- Specific testing framework selection (Jest, Mocha, etc.)
- Code coverage thresholds and reporting requirements

Exceptions:
- EXC-001: Legacy codebases undergoing gradual migration may temporarily maintain mixed test organization
- EXC-002: Prototype or spike projects with limited lifespan (under 2 weeks) may use simplified test organization

## Rationale

- Pattern detected across 9 test files with 92.37% confidence indicates this is an established and consistent practice in the codebase
- Separation of unit and integration tests enables targeted test execution, reducing CI/CD feedback time and development iteration cycles
- Mocking in unit tests ensures tests remain fast, deterministic, and focused on single-component behavior rather than system-wide interactions
- Clear test organization improves maintainability by making test purpose and scope immediately apparent from file location and structure

## Consequences

Positive:
- Faster CI/CD pipelines through ability to run quick unit tests first and gate slower integration tests
- Improved developer experience with rapid feedback during local development when running unit tests only
- Clearer test failure diagnostics as unit test failures indicate component-level issues while integration test failures indicate interaction problems
- Better test maintainability through consistent organization and clear separation of concerns

Negative:
- Requires discipline to correctly classify tests as unit vs integration, with potential for misclassification
- Additional overhead in maintaining mock implementations for unit tests, which must stay synchronized with real implementations
- Risk of over-mocking in unit tests leading to tests that pass but don't reflect real-world behavior
- May require refactoring of existing test suites to comply with the organizational structure

## Alternatives

- Flat test directory structure with all tests in a single 'tests' folder without unit/integration separation (rejected)
  Rejected because: Makes it impossible to run targeted test subsets efficiently and obscures test purpose and scope
  When valid: Only appropriate for very small projects with fewer than 10 test files
- Co-locate tests with source files (e.g., Component.ts and Component.test.ts in same directory) (rejected)
  Rejected because: Conflicts with detected pattern and makes it harder to distinguish unit from integration tests
  When valid: May be appropriate for component libraries where tests are tightly coupled to individual exports
- Three-tier structure with unit, integration, and e2e directories (deferred)
  Rejected because: Not rejected, but current evidence shows only unit/integration split; e2e may be added later
  When valid: Should be adopted when end-to-end testing requirements emerge requiring browser automation or full system deployment

## Risks

- Developers may incorrectly classify integration tests as unit tests, leading to slow unit test suites
  Mitigation: Provide clear guidelines and examples; implement CI checks for unit test execution time thresholds
  Owner: Engineering team leads
- Over-reliance on mocking in unit tests may create false confidence when mocks diverge from real implementations
  Mitigation: Maintain comprehensive integration test coverage; use contract testing where appropriate; regularly review mock accuracy
  Owner: QA and engineering teams
- Refactoring existing test suites to comply may introduce regressions or reduce coverage temporarily
  Mitigation: Implement gradual migration with coverage tracking; require no coverage decrease during refactoring
  Owner: Development team

## Implementation Notes

- Create 'tests/unit' and 'tests/integration' directories if they don't exist; move existing tests to appropriate locations based on their scope
- Update package.json scripts to support running unit and integration tests separately (e.g., 'npm run test:unit', 'npm run test:integration')
- Configure test framework to use appropriate mocking libraries (e.g., jest.mock, sinon) and establish patterns for common mocking scenarios
- Document clear criteria for classifying tests: unit tests test a single module in isolation with mocked dependencies; integration tests test multiple modules or external systems together
- Update CI/CD pipeline configuration to run unit tests in an early stage and integration tests in a later stage, with appropriate failure handling

## Continuation Context


Verify commands:
- find tests -type f -name '*.test.*' | grep -E '^tests/(unit|integration)/' | wc -l
- grep -r 'jest.mock\|sinon\|mock' tests/unit/ | wc -l
- test -d tests/unit && test -d tests/integration && echo 'Test directories exist' || echo 'Missing test directories'

Accept when:
- All test files are located in either tests/unit or tests/integration directories with no test files in the root tests directory
- Unit test files contain evidence of mocking (jest.mock, sinon, or similar mocking constructs)
- CI/CD pipeline configuration shows separate execution stages or commands for unit and integration tests

## Enforcement

- Verified by: Automated CI/CD pipeline checks verify test directory structure compliance
- Verified by: Code review process validates new tests are placed in appropriate directories
- Verified by: Static analysis tools check for presence of mocking in unit tests
- Violation handling: CI pipeline fails if test files are found outside unit/integration directories
- Violation handling: Pull requests with incorrectly classified tests receive automated comments with guidance
- Violation handling: Quarterly audits identify and remediate test organization violations
- Exception process: Request exception through tech lead with documented justification
- Exception process: Include exception rationale in test file header comment
- Exception process: Track exceptions in architectural decision log for periodic review