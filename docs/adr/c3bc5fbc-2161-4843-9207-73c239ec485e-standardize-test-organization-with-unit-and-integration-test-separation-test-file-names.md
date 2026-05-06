# Standardize Test Organization with Unit and Integration Test Separation: Test File Names

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all test code and CI/CD pipeline configurations within the codebase.

## Context

- The codebase contains 9 test files demonstrating a consistent pattern of separating unit tests from integration tests in distinct directory structures
- Test files are organized under tests/unit/ and tests/integration/ directories, covering core functionality (apply-engine, revert-engine, RuleProcessor), agent implementations (FirebenderAgent, revert-agents, skills-mcp-agent), and feature testing (hierarchical-rules, constants)
- This organizational pattern enables different testing strategies, execution speeds, and isolation levels appropriate to each test type
- The pattern supports CI/CD pipelines that can selectively run fast unit tests for rapid feedback and comprehensive integration tests for deployment validation
- The separation facilitates mocking strategies where unit tests can isolate components while integration tests verify end-to-end behavior

## Problem Statement

Without a standardized approach to organizing unit and integration tests, teams face challenges in maintaining consistent test execution strategies, optimizing CI/CD pipeline performance, and ensuring appropriate test isolation. Mixed test organization leads to slower feedback loops, difficulty in identifying test failures, and inconsistent mocking practices across the codebase.

## Decision

1. SHOULD: Test file names SHOULD follow the pattern {component-name}.test.{ts|js} to clearly identify the component being tested

## Policy Block

- SHOULD Test file names SHOULD follow the pattern {component-name}.test.{ts|js} to clearly identify the component being tested

In scope:
- All test files for application code including core functionality, agents, utilities, and features
- Test organization for both TypeScript (.ts) and JavaScript (.js) test files
- CI/CD pipeline test execution stages and strategies
- Test mocking and isolation strategies

Out of scope:
- Third-party library tests or vendor code testing
- Documentation examples that are not executable tests
- Prototype or experimental code in non-production branches
- Build scripts or tooling configuration files

Exceptions:
- EX-001: Legacy test files exist in a different structure and are being gradually migrated
- EX-002: A test genuinely spans both unit and integration concerns and cannot be reasonably split

## Rationale

- The detected pattern shows consistent adoption across 9 files with 92.37% confidence, indicating this is an established and successful practice in the codebase
- Separating unit and integration tests enables optimized CI/CD pipelines where fast unit tests provide immediate feedback while comprehensive integration tests validate system behavior
- Clear directory structure makes it immediately obvious what type of test is being written or modified, reducing cognitive load and onboarding time
- The pattern supports different testing strategies: unit tests focus on isolated component behavior with mocking, while integration tests verify real component interactions

## Consequences

Positive:
- Faster CI/CD feedback loops by running quick unit tests first before slower integration tests
- Clear mental model for developers about where to place new tests and what testing strategy to employ
- Easier to identify and debug test failures based on test type and location
- Supports parallel test execution strategies where unit and integration tests can run in separate CI jobs
- Facilitates selective test execution during development (e.g., running only unit tests for rapid iteration)

Negative:
- Requires discipline to correctly categorize tests as unit vs integration, which can be subjective in edge cases
- May lead to code duplication if similar setup is needed across unit and integration tests
- Adds directory structure complexity compared to a flat test organization
- Developers must understand the distinction between unit and integration tests to place files correctly

## Alternatives

- Co-locate tests with source code (e.g., component.ts and component.test.ts in same directory) (rejected)
  Rejected because: Makes it difficult to run different test types separately in CI/CD pipelines and obscures the distinction between unit and integration testing strategies
  When valid: May be appropriate for small libraries or packages with minimal integration testing needs
- Use a single tests/ directory with naming conventions (e.g., *.unit.test.ts, *.integration.test.ts) (rejected)
  Rejected because: Relies on naming discipline rather than enforced structure, makes glob patterns more complex, and provides less visual clarity in file explorers
  When valid: Could work for very small projects with fewer than 20 test files
- Organize tests by feature rather than by test type (rejected)
  Rejected because: Makes it harder to execute all unit tests or all integration tests independently, complicating CI/CD pipeline optimization
  When valid: Appropriate for feature-based monorepos where each feature is independently deployable

## Risks

- Developers may incorrectly categorize tests, placing integration tests in unit/ or vice versa, leading to pipeline inefficiencies
  Mitigation: Provide clear documentation and examples of unit vs integration tests; implement code review checklist items; consider automated checks for common anti-patterns (e.g., unit tests with network calls)
  Owner: Engineering team and tech leads
- Test execution time may grow significantly if integration tests are not properly optimized or parallelized
  Mitigation: Monitor test execution times in CI/CD; implement test parallelization; set time budgets for test suites; regularly review and optimize slow tests
  Owner: DevOps and engineering team
- Overly strict separation may lead to gaps in test coverage where neither unit nor integration tests adequately cover certain scenarios
  Mitigation: Maintain code coverage metrics across both test types; conduct regular test coverage reviews; allow exceptions for hybrid tests when justified
  Owner: Engineering team

## Implementation Notes

- Create tests/unit/ and tests/integration/ directories at the project root if they don't exist
- Mirror the source code directory structure within tests/unit/ (e.g., src/core/engine.ts → tests/unit/core/engine.test.ts)
- Configure test runners (Jest, Mocha, etc.) with separate commands or scripts for unit and integration tests using glob patterns
- Update CI/CD pipeline configuration to run unit tests in an early stage and integration tests in a later stage
- Document the distinction between unit and integration tests in the project's testing guidelines with concrete examples from the codebase
- Consider adding linting rules or pre-commit hooks to validate test file placement

## Continuation Context


Verify commands:
- find tests/unit -name '*.test.*' -type f | wc -l
- find tests/integration -name '*.test.*' -type f | wc -l
- grep -r 'tests/unit\|tests/integration' package.json jest.config.js .github/workflows/*.yml 2>/dev/null || echo 'Check CI config'

Accept when:
- Both tests/unit/ and tests/integration/ directories exist and contain test files
- Test runner configuration includes separate commands or glob patterns for unit and integration tests
- CI/CD pipeline configuration shows distinct stages or jobs for unit and integration test execution
- At least 80% of test files follow the naming convention {component}.test.{ts|js}

## Enforcement

- Verified by: Automated CI/CD pipeline checks that verify test directory structure
- Verified by: Code review process with explicit checklist item for test placement
- Verified by: Static analysis or linting rules that validate test file locations
- Verified by: Periodic architecture reviews examining test organization patterns
- Violation handling: CI pipeline warnings for tests placed outside standard directories
- Violation handling: Code review feedback requesting relocation of misplaced tests
- Violation handling: Automated PR comments suggesting correct test placement based on file path analysis
- Violation handling: Quarterly technical debt reviews to identify and remediate test organization violations
- Exception process: Developer documents the exception rationale in the test file header or PR description
- Exception process: Tech lead or senior engineer reviews and approves the exception during code review
- Exception process: Exception is tracked in a central registry (wiki page, ADR, or issue tracker) with justification
- Exception process: Exceptions are reviewed quarterly to determine if they can be resolved or if the ADR needs updating