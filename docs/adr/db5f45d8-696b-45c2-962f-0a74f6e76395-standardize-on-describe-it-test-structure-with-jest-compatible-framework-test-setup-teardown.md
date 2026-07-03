# Standardize on describe/it Test Structure with Jest-Compatible Framework: Test Setup Teardown

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all test files in the codebase. Test authors MUST follow the prescribed structure when writing unit, integration, or end-to-end tests.

## Context

- The codebase contains 10 test files demonstrating consistent use of describe/it block structure for organizing test suites and individual test cases
- Test files span multiple testing concerns including MCP server configuration, agent behavior, backup prevention, integration workflows, and skills propagation
- Tests use beforeAll, beforeEach, afterAll, and afterEach lifecycle hooks for setup and teardown, indicating a need for consistent test isolation patterns
- Evidence shows extensive use of JSON.parse for validating configuration files, file system operations via fs/promises, and path manipulation for cross-platform compatibility
- Integration tests demonstrate complex multi-step workflows (init → setup → apply → verify) requiring structured test organization to maintain readability and debuggability

## Problem Statement

Without a standardized test structure, test suites become difficult to navigate, maintain, and extend. Inconsistent organization patterns lead to unclear test intent, poor isolation between test cases, and challenges in debugging failures. The codebase requires a uniform approach to test organization that supports both simple unit tests and complex integration scenarios while maintaining clear hierarchical relationships between test suites and cases.

## Decision

1. MUST: Test setup and teardown logic MUST use appropriate lifecycle hooks: beforeAll for suite-level setup, beforeEach for test-level setup, afterAll for suite-level cleanup, and afterEach for test-level cleanup

## Policy Block

- MUST Test setup and teardown logic MUST use appropriate lifecycle hooks: beforeAll for suite-level setup, beforeEach for test-level setup, afterAll for suite-level cleanup, and afterEach for test-level cleanup

In scope:
- All unit test files in tests/unit/**/*.test.ts
- All integration test files in tests/integration/**/*.test.ts and tests/e2e/**/*.test.ts
- All feature-specific test files in tests/**/*.test.ts
- Test harness and utility functions that support test execution

Out of scope:
- Build scripts and configuration files (jest.config.js, tsconfig.json)
- Test fixtures and mock data files
- Production source code in src/ directory
- Documentation and example files

Exceptions:
- EXC-001: Legacy test files written before this ADR was adopted may temporarily use alternative structures
- EXC-002: Performance benchmark tests may use alternative structures optimized for timing measurements

## Rationale

- The evidence shows 10 test files with 92.66% confidence all using describe/it structure, indicating this is the established pattern in the codebase
- Consistent test organization improves developer productivity by making test intent immediately clear and reducing cognitive load when navigating unfamiliar test suites
- The describe/it pattern is widely adopted in the JavaScript/TypeScript ecosystem and supported by major testing frameworks (Jest, Vitest, Mocha), ensuring tooling compatibility and developer familiarity
- Lifecycle hooks (beforeAll, afterAll, beforeEach, afterEach) provide deterministic setup and teardown execution order, critical for test isolation in integration scenarios involving file system operations and temporary directories

## Consequences

Positive:
- Uniform test structure across the codebase improves maintainability and reduces onboarding time for new contributors
- Clear hierarchical organization with describe blocks enables better test filtering and selective execution during development
- Consistent use of lifecycle hooks ensures proper resource cleanup and prevents test pollution that can cause flaky test failures
- Descriptive test names in it blocks serve as living documentation of expected system behavior

Negative:
- Developers must learn and follow the prescribed structure, adding a small learning curve for those unfamiliar with describe/it patterns
- Deeply nested describe blocks can become verbose and may require refactoring to maintain readability
- Strict adherence to lifecycle hooks may add boilerplate code for simple tests that don't require complex setup/teardown

## Alternatives

- Use flat test structure with test() function and no describe blocks (rejected)
  Rejected because: Flat structure lacks hierarchical organization needed for complex integration tests with multiple related scenarios. Evidence shows tests like 'ruler.integration.test.ts' require nested organization to group related verification steps.
  When valid: May be acceptable for very simple unit tests with 1-2 test cases, but not recommended as a general pattern
- Use class-based test organization with methods as test cases (rejected)
  Rejected because: Class-based approach is less idiomatic in JavaScript/TypeScript testing ecosystem and lacks first-class support in Jest. Evidence shows no usage of class-based test patterns in the 10 analyzed files.
  When valid: Could be considered for projects using testing frameworks that specifically support class-based tests (e.g., older JUnit-style frameworks)
- Mix describe/it with other organizational patterns on a per-file basis (rejected)
  Rejected because: Inconsistent patterns across files increase cognitive load and make it harder to establish team conventions. Standardization provides clear benefits for collaboration.
  When valid: Never valid within this codebase; consistency is a core requirement

## Risks

- Existing test files not following this pattern may require refactoring, consuming development time
  Mitigation: Conduct audit of all test files, prioritize refactoring based on test criticality and modification frequency. Allow gradual migration with documented exceptions.
  Owner: Testing team lead
- Over-nesting of describe blocks can lead to deeply indented code that is hard to read
  Mitigation: Establish guideline of maximum 3 levels of nesting. Use test file splitting or helper functions to reduce complexity when deeper nesting is needed.
  Owner: Code reviewers
- Improper use of lifecycle hooks (e.g., using beforeAll when beforeEach is needed) can cause test pollution and flaky failures
  Mitigation: Provide clear documentation and examples of when to use each lifecycle hook. Include test isolation checks in CI pipeline.
  Owner: Engineering team

## Implementation Notes

- Use describe blocks to group tests by feature, component, or behavior. Name describe blocks with clear, descriptive strings (e.g., 'MCP Backup Prevention for All Agents', 'Gemini MCP key usage')
- Write it block descriptions as complete sentences starting with 'should' to clearly state expected behavior (e.g., 'should handle idempotent re-runs without creating backup files')
- Place shared setup logic in beforeAll or beforeEach hooks. Use beforeAll for expensive operations that can be shared across tests (e.g., creating test projects). Use beforeEach for test-specific state that must be isolated.
- Always pair setup hooks with corresponding cleanup hooks (beforeAll with afterAll, beforeEach with afterEach) to ensure resources are properly released
- For integration tests with multiple verification steps, consider using nested describe blocks to organize related assertions while keeping individual it blocks focused on single concerns
- When tests require custom timeouts (e.g., for long-running integration tests), specify timeout as third parameter: it('test name', async () => { ... }, 60000)

## Continuation Context


Verify commands:
- grep -r "describe(" tests/ | wc -l
- grep -r "it(" tests/ | wc -l
- grep -r "beforeAll\|beforeEach\|afterAll\|afterEach" tests/ | wc -l
- npm test -- --listTests | xargs grep -L "describe(" || echo 'All test files use describe blocks'

Accept when:
- All test files in tests/ directory contain at least one describe block organizing test cases
- All test cases are defined using it blocks with descriptive names
- Test files using temporary resources (file system, network) include appropriate cleanup in afterAll or afterEach hooks
- Code review confirms new test files follow the prescribed structure before merge

## Enforcement

- Verified by: Automated linting rules checking for presence of describe/it blocks in .test.ts files
- Verified by: Code review checklist item verifying test structure compliance
- Verified by: CI pipeline test execution confirming all tests run successfully with proper isolation
- Violation handling: Linter warnings for test files missing describe blocks
- Violation handling: Code review feedback requesting restructuring before approval
- Violation handling: Test failures due to improper cleanup trigger investigation and remediation
- Exception process: Developer documents rationale for exception in test file header comment
- Exception process: Tech lead or testing team lead reviews and approves exception
- Exception process: Exception is tracked in team documentation with migration plan if temporary