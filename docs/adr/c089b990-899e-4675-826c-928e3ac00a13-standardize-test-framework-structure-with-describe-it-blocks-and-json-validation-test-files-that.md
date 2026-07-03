# Standardize Test Framework Structure with describe/it Blocks and JSON Validation: Test Files That

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all test code in the codebase.

## Context

- The codebase contains 13 test files that consistently use describe/it block structure for organizing test suites, indicating a standardized testing approach across the project
- All test files process JSON configuration data from multiple sources (.ruler/mcp.json, .vscode/mcp.json, .cursor/mcp.json, .gemini/settings.json) requiring consistent parsing and validation patterns
- Tests validate MCP (Model Context Protocol) server configurations across multiple AI agent integrations (Gemini, Copilot, Cursor, Claude, Firebase, OpenHands, AugmentCode) with different native configuration formats
- The test suite uses a common harness (setupTestProject, teardownTestProject, runRuler) to create isolated test environments and verify configuration file operations including backup prevention and idempotency
- Security-critical input validation through JSON.parse operations appears consistently across all test files, requiring standardized error handling and validation patterns

## Problem Statement

Test code that processes external configuration files and validates JSON data structures requires consistent organization, validation patterns, and security practices to prevent parsing errors, injection vulnerabilities, and inconsistent test behavior across multiple agent integration scenarios.

## Decision

1. MUST: Test files that read configuration files MUST validate the structure and required properties of parsed JSON objects before making assertions

## Policy Block

- MUST Test files that read configuration files MUST validate the structure and required properties of parsed JSON objects before making assertions

In scope:
- All test files in the tests/ directory
- Unit tests for agent implementations (tests/unit/agents/)
- Integration tests for MCP configuration (tests/apply-mcp.*.test.ts)
- End-to-end tests (tests/e2e/)
- Test harness and utility code (tests/harness.ts)

Out of scope:
- Production source code in src/ directory
- Build scripts and tooling configuration
- Documentation and example code
- Third-party library code

Exceptions:
- EXC-001: Legacy test files written before this standard was established
- EXC-002: Performance tests or stress tests where describe/it overhead is measurable

## Rationale

- The consistent use of describe/it blocks across 13 test files with 93.30% confidence indicates an established pattern that improves test readability and maintainability
- JSON.parse operations on configuration files from multiple sources (.ruler/, .vscode/, .cursor/, .gemini/, .idx/) require standardized validation to prevent security vulnerabilities from malformed or malicious input
- The test harness pattern (setupTestProject, teardownTestProject) provides isolation and prevents test pollution, which is critical for tests that modify filesystem state and configuration files
- Standardized test structure enables better test reporting, easier debugging, and consistent behavior across different test runners and CI environments

## Consequences

Positive:
- Improved test readability and maintainability through consistent structure and naming conventions
- Reduced risk of security vulnerabilities from unvalidated JSON parsing in test code
- Better test isolation and reliability through standardized setup/teardown patterns
- Easier onboarding for new developers who can quickly understand test organization and patterns
- Consistent error handling across test files reduces flaky tests and improves CI reliability

Negative:
- Additional boilerplate code required for describe/it blocks and setup/teardown hooks
- Learning curve for developers unfamiliar with BDD-style test structure
- Potential performance overhead from setup/teardown hooks in large test suites
- Refactoring effort required to bring existing non-compliant tests into alignment

## Alternatives

- Use flat test functions without describe/it structure (rejected)
  Rejected because: Flat structure lacks organization and makes it difficult to group related tests, understand test scope, and generate meaningful test reports
  When valid: Only for very simple test files with 1-2 test cases
- Use schema validation libraries (e.g., Zod, Joi) instead of manual JSON validation (deferred)
  Rejected because: Would require additional dependencies and migration effort; current JSON.parse with validation is sufficient for test code
  When valid: If production code adopts schema validation, tests should follow the same pattern
- Allow each test file to define its own setup/teardown patterns (rejected)
  Rejected because: Inconsistent patterns across test files lead to maintenance burden and increased risk of test pollution
  When valid: Only when test requirements are fundamentally incompatible with the standard harness

## Risks

- Malformed JSON in test fixtures could cause unhandled exceptions and test failures that are difficult to debug
  Mitigation: Implement try-catch blocks around JSON.parse operations with descriptive error messages; validate JSON structure before assertions
  Owner: Engineering team
- Test isolation failures could occur if setup/teardown hooks are not properly implemented, leading to flaky tests
  Mitigation: Use the standardized test harness (setupTestProject, teardownTestProject) which handles cleanup; verify isolation in CI
  Owner: Engineering team
- Performance degradation in large test suites due to setup/teardown overhead
  Mitigation: Use beforeAll/afterAll for expensive setup when possible; monitor test execution time in CI; optimize harness utilities
  Owner: Engineering team

## Implementation Notes

- Use the existing test harness utilities (setupTestProject, teardownTestProject, runRuler) from tests/harness.ts for consistent environment management
- Wrap JSON.parse operations in try-catch blocks or use helper functions that provide better error messages for debugging
- Structure describe blocks hierarchically: top-level for the component/feature, nested for specific scenarios or behaviors
- Use descriptive test names that read as specifications: 'should not create backup files when MCP is handled correctly' rather than 'test backup'
- Import core libraries (fs/promises, path) consistently and use async/await for file operations to avoid callback complexity

## Continuation Context


Verify commands:
- grep -r "describe(" tests/ | wc -l
- grep -r "it(" tests/ | wc -l
- grep -r "JSON.parse" tests/ --include="*.test.ts" | wc -l
- npm test -- --coverage

Accept when:
- All test files in tests/ directory use describe/it block structure
- All JSON.parse operations in test files are wrapped with error handling or validation
- Test suite passes with 100% success rate in CI environment
- Code coverage for test harness utilities is above 80%

## Enforcement

- Verified by: Automated linting rules that detect test files without describe/it structure
- Verified by: Code review checklist requiring verification of JSON validation in tests
- Verified by: CI pipeline that runs test suite and reports on test structure compliance
- Verified by: Static analysis tools that detect unhandled JSON.parse operations
- Violation handling: CI build fails if tests do not follow describe/it structure
- Violation handling: Code review blocks merge if JSON parsing lacks error handling
- Violation handling: Automated comments on pull requests identifying non-compliant test patterns
- Violation handling: Quarterly audit of test files to identify and remediate violations
- Exception process: Developer submits exception request with justification to tech lead
- Exception process: Tech lead reviews and approves/rejects based on technical merit
- Exception process: Approved exceptions are documented in test file comments with EXC-ID reference
- Exception process: Exceptions are reviewed quarterly and may be revoked if circumstances change