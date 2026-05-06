# Standardize Unit and Integration Test Organization in tests/ Directory: Integration Tests Placed

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all test file creation and organization within the codebase.

## Context

- The codebase contains 30 test files following a consistent organizational pattern with tests/unit/ and tests/integration/ directories, indicating a deliberate architectural decision to separate test types
- Test files are organized by component type (agents, core) and test scope (unit vs integration), suggesting a need for maintainability and clear test execution boundaries
- The pattern shows TypeScript test files (.test.ts) across multiple agent implementations (ClaudeAgent, RooCodeAgent, OpenHandsAgent, KiroAgent, QwenCodeAgent, TraeAgent, MistralVibeAgent) and core functionality
- Integration tests focus on cross-cutting concerns (unified-equivalence, skills-mcp, skills-config-precedence) while unit tests target isolated component behavior
- The high confidence (91.85%) and support count (30 files) indicates this is an established, consistently-applied pattern rather than an emerging practice

## Problem Statement

Without a standardized test organization structure, test files become scattered across the codebase, making it difficult to run specific test suites (unit vs integration), understand test scope, maintain consistent CI/CD pipelines, and ensure appropriate test isolation. This leads to slower test execution, unclear test boundaries, and increased maintenance overhead.

## Decision

1. MUST: All integration tests MUST be placed in the tests/integration/ directory hierarchy

## Policy Block

- MUST All integration tests MUST be placed in the tests/integration/ directory hierarchy

In scope:
- All TypeScript test files (.test.ts)
- Unit tests for agents, core functionality, and utilities
- Integration tests for cross-component workflows
- Regression tests and bug reproduction tests

Out of scope:
- End-to-end (e2e) tests which may have their own directory structure
- Performance or load tests which may require separate organization
- Test fixtures, mocks, or test utilities (these may live in tests/fixtures/ or tests/helpers/)
- Documentation or example files that are not executable tests

Exceptions:
- EXC-001: Bug reproduction tests that don't fit cleanly into unit or integration categories
- EXC-002: Temporary test files for experimental features or spikes

## Rationale

- The pattern detection identified 30 files with 91.85% confidence, demonstrating this is an established and consistently-applied organizational standard in the codebase
- Separating unit and integration tests enables selective test execution in CI/CD pipelines, allowing fast feedback loops with unit tests while running slower integration tests less frequently
- Clear directory structure improves developer experience by making it immediately obvious where to find and add tests, reducing cognitive load and onboarding time
- The organization by component type (agents, core) mirrors the source code structure, making it intuitive to locate tests for specific functionality

## Consequences

Positive:
- CI/CD pipelines can selectively run unit tests for fast feedback and integration tests for comprehensive validation
- Developers can quickly locate relevant tests by following the predictable directory structure
- Test execution time is optimized by running isolated unit tests separately from integration tests
- Code coverage tools can generate separate reports for unit vs integration test coverage
- New team members can easily understand test organization and contribute tests following established patterns

Negative:
- Requires discipline to correctly classify tests as unit vs integration, which may be ambiguous for some test cases
- Adds directory depth which may make file paths longer in test output and stack traces
- Refactoring code structure may require corresponding test directory reorganization
- Edge cases like bug reproduction tests may not fit cleanly into the binary unit/integration classification

## Alternatives

- Co-locate tests with source code (e.g., src/agents/ClaudeAgent.test.ts next to ClaudeAgent.ts) (rejected)
  Rejected because: Mixing test and source code complicates build processes, increases bundle size risk, and makes it harder to exclude tests from production builds. The current pattern provides cleaner separation.
  When valid: May be appropriate for very small projects or libraries where test proximity to source is highly valued
- Flat test directory structure (all tests in tests/ without subdirectories) (rejected)
  Rejected because: Does not scale well with 30+ test files and growing. Makes it impossible to selectively run unit vs integration tests and loses organizational benefits of mirroring source structure.
  When valid: Only suitable for very small projects with fewer than 10 test files
- Three-tier structure with tests/unit/, tests/integration/, and tests/e2e/ (deferred)
  Rejected because: Not rejected, but no e2e tests detected in current pattern. This could be adopted in the future if end-to-end testing is introduced.
  When valid: Should be adopted when end-to-end tests are added to the test suite

## Risks

- Developers may incorrectly classify tests, placing integration tests in tests/unit/ or vice versa, leading to incorrect test execution in CI/CD
  Mitigation: Provide clear documentation and examples of unit vs integration test characteristics. Implement linting rules to detect common anti-patterns (e.g., unit tests with network calls).
  Owner: Engineering team, enforced through code review
- Test file paths may become very long with deep directory nesting, causing issues on Windows systems with path length limits
  Mitigation: Monitor path lengths and keep subdirectory nesting to a maximum of 3-4 levels. Use concise but descriptive directory names.
  Owner: Engineering team
- Refactoring source code structure requires corresponding test reorganization, which may be forgotten or done inconsistently
  Mitigation: Include test file location updates in refactoring checklists. Use automated tools to detect orphaned or misplaced test files.
  Owner: Engineering team, enforced through code review

## Implementation Notes

- Use test runner configuration (e.g., Jest, Vitest) to define separate test patterns: 'tests/unit/**/*.test.ts' for unit tests and 'tests/integration/**/*.test.ts' for integration tests
- Create npm scripts or make targets for running test suites separately: 'npm run test:unit' and 'npm run test:integration'
- Mirror the source code directory structure within tests/unit/ (e.g., src/agents/ClaudeAgent.ts → tests/unit/agents/ClaudeAgent.test.ts)
- For bug reproduction tests that don't fit the unit/integration dichotomy, place them in tests/ root with a descriptive name indicating the issue (e.g., tests/corrupted-marker-fix.test.ts, tests/gemini-no-type-field.test.ts)

## Continuation Context


Verify commands:
- find tests/unit -name '*.test.ts' | wc -l
- find tests/integration -name '*.test.ts' | wc -l
- grep -r 'describe\|it\|test' tests/unit tests/integration | grep -v node_modules

Accept when:
- All test files with .test.ts extension are located in either tests/unit/, tests/integration/, or tests/ root (for special cases)
- Test runner can successfully execute unit and integration tests separately using path-based patterns
- Directory structure within tests/unit/ mirrors the source code organization in src/

## Enforcement

- Verified by: CI/CD pipeline checks that verify test files are in correct directories
- Verified by: Code review process ensures new tests follow organizational structure
- Verified by: Automated linting rules that flag test files in incorrect locations
- Violation handling: CI build fails if test files are found outside approved directories (tests/unit/, tests/integration/, tests/)
- Violation handling: Pull requests with misplaced test files are flagged in code review and must be corrected before merge
- Violation handling: Quarterly audits identify and relocate any misplaced test files
- Exception process: Developer documents reason for exception in PR description and test file comment
- Exception process: Tech lead or architect reviews and approves exception during code review
- Exception process: Exception is documented in ADR exceptions log with justification and expiration date if temporary