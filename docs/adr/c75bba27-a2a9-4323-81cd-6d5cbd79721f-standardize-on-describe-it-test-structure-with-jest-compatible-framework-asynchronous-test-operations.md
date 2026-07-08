# Standardize on describe/it Test Structure with Jest-Compatible Framework: Asynchronous Test Operations

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains 28 test files demonstrating consistent use of describe/it block structure for organizing test suites and individual test cases
- Test files span unit tests (agents, lowercase identifiers), integration tests (unified config loader, skills MCP, Gemini integration), and functional tests (MCP configuration, TOML disable scenarios)
- Tests validate agent-specific behavior, configuration merging, MCP server definitions, backup file handling, and skills propagation across multiple agent implementations
- The testing approach supports both synchronous and asynchronous test execution with beforeEach/afterEach lifecycle hooks for test isolation
- Test files consistently import core libraries (fs/promises, path, os, child_process) and perform JSON parsing for configuration validation and assertion

## Problem Statement

Without a standardized test framework and structure, test suites become inconsistent in organization, lifecycle management, and assertion patterns, making it difficult to maintain test quality, onboard new contributors, and ensure reliable verification of agent behavior, configuration merging, and MCP server integration across the codebase.

## Decision

1. MUST: Asynchronous test operations MUST use async/await syntax with proper error handling

## Policy Block

- MUST Asynchronous test operations MUST use async/await syntax with proper error handling

In scope:
- All TypeScript test files in tests/ directory
- Unit tests for agent implementations
- Integration tests for configuration loading and merging
- Functional tests for MCP server definitions and TOML configuration
- Test harness utilities and helper functions

Out of scope:
- End-to-end tests that require external service dependencies
- Performance benchmarking tests with specialized timing requirements
- Manual testing procedures documented outside the test suite
- Third-party library test files in node_modules

Exceptions:
- EXC-001: Legacy test files written before this ADR adoption may temporarily use alternative test structures

## Rationale

- The evidence shows 28 test files with 92.70% confidence using describe/it structure, indicating strong organic adoption and team familiarity with this pattern
- Consistent test structure enables developers to quickly locate test cases, understand test organization, and add new tests following established patterns
- The use of lifecycle hooks (beforeEach, afterEach, beforeAll, afterAll) across integration tests demonstrates the need for proper resource management and test isolation
- Jest-compatible syntax provides access to a mature ecosystem of matchers, mocking utilities, and IDE integrations while maintaining compatibility with alternative test runners

## Consequences

Positive:
- Uniform test structure reduces cognitive load when navigating between test files and accelerates test authoring
- Lifecycle hooks ensure proper setup and teardown, preventing test pollution and resource leaks in integration tests
- Hierarchical describe blocks enable logical grouping of related test cases, improving test discoverability and reporting
- Jest-compatible syntax provides access to rich assertion libraries, snapshot testing, and mocking capabilities

Negative:
- Developers unfamiliar with describe/it syntax require onboarding time to understand the test structure conventions
- Deeply nested describe blocks can become difficult to read if overused, requiring discipline in test organization
- Asynchronous lifecycle hooks add complexity to test execution order and error handling
- Migration of any existing non-conforming tests requires engineering effort and regression validation

## Alternatives

- Use flat test structure with test() function and no describe blocks (rejected)
  Rejected because: Flat structure lacks hierarchical organization needed for complex test suites with multiple agent implementations and configuration scenarios, as evidenced by nested describe blocks in agent-specific and integration tests
  When valid: May be acceptable for simple utility function tests with fewer than 5 test cases
- Adopt TAP (Test Anything Protocol) with tape or node-tap (rejected)
  Rejected because: TAP syntax diverges from the established describe/it pattern already adopted across 28 test files, requiring significant migration effort without clear benefit
  When valid: Could be considered for CLI tool testing where TAP output format provides specific advantages
- Use class-based test organization with decorators (rejected)
  Rejected because: Class-based approach adds boilerplate and complexity compared to functional describe/it blocks, and is not evidenced in the existing test suite
  When valid: May be appropriate for test suites requiring complex shared state or inheritance hierarchies

## Risks

- Test execution order dependencies may emerge if lifecycle hooks are not properly isolated, causing flaky tests
  Mitigation: Enforce test isolation by ensuring each test case can run independently; use linting rules to detect shared mutable state; document lifecycle hook best practices
  Owner: Engineering team
- Asynchronous test operations without proper timeout configuration may cause tests to hang indefinitely
  Mitigation: Configure global test timeout in test framework configuration; use explicit timeout parameters for long-running integration tests; implement timeout guards in test harness utilities
  Owner: Engineering team
- Inconsistent assertion library usage across test files may reduce test readability and maintainability
  Mitigation: Document standard assertion patterns in testing guidelines; provide test templates for common scenarios (agent tests, config tests, MCP tests); conduct code review for assertion consistency
  Owner: Engineering team

## Implementation Notes

- Use setupTestProject and teardownTestProject harness utilities for integration tests requiring temporary project structures, as evidenced in agent-specific and MCP configuration tests
- Import core libraries (fs/promises, path, os) at the top of test files for consistent file system operations and path manipulation
- Use JSON.parse for configuration validation in tests that verify MCP server definitions, agent settings, and TOML-to-JSON transformations
- Organize test files by type: unit tests in tests/unit/, integration tests in tests/integration/, with descriptive filenames matching the component under test
- Use runRuler or runRulerWithInheritedStdio helper functions for tests that invoke the ruler CLI, ensuring consistent command execution and output capture

## Continuation Context


Verify commands:
- grep -r "describe(" tests/ | wc -l
- grep -r "it(" tests/ | wc -l
- grep -r "beforeEach\|afterEach\|beforeAll\|afterAll" tests/ | wc -l
- npm test -- --listTests | grep -E '\.test\.ts$' | wc -l

Accept when:
- All test files in tests/ directory use describe blocks for test suite organization
- All individual test cases are defined using it blocks with descriptive names
- Test files requiring setup/teardown use appropriate lifecycle hooks (beforeEach, afterEach, beforeAll, afterAll)
- Test suite executes successfully with no hanging tests or resource leaks

## Enforcement

- Verified by: Code review process checks for describe/it structure in new test files
- Verified by: CI pipeline executes full test suite and reports test structure violations
- Verified by: ESLint rules enforce test naming conventions and lifecycle hook usage
- Violation handling: Pull requests with non-conforming test structure receive review comments requesting updates
- Violation handling: CI pipeline fails if tests do not follow describe/it structure (enforceable via custom linting rules)
- Violation handling: Existing non-conforming tests are tracked in technical debt backlog with migration priority
- Exception process: Developer documents rationale for exception in test file header comment
- Exception process: Engineering team lead reviews and approves exception with documented justification
- Exception process: Exception is recorded in ADR exceptions log with expiration date or migration plan