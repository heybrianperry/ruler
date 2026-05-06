# Standardize Test File Organization in tests/ Directory with Unit and Integration Separation: Core Functionality Tests

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all test file creation and organization activities within the codebase.

## Context

- The codebase contains 30 test files following a consistent organizational pattern with clear separation between unit tests (tests/unit/) and integration tests (tests/integration/)
- Test files are organized by functional area including core functionality tests, agent-specific tests, and integration tests for skills and configuration
- The pattern shows systematic testing of multiple agent implementations (ClaudeAgent, RooCodeAgent, OpenHandsAgent, KiroAgent, QwenCodeAgent, TraeAgent, MistralVibeAgent, AgentAdapters) indicating a multi-agent architecture
- Test organization includes edge case handling, corrupted data scenarios, and unified equivalence testing, suggesting mature test coverage practices
- The consistent .test.ts extension and directory structure indicates TypeScript-based testing with established conventions

## Problem Statement

Without a standardized test organization structure, test files can become scattered across the codebase, making it difficult to distinguish between unit tests, integration tests, and other test types. This leads to inconsistent test execution strategies, unclear CI/CD pipeline configuration, and difficulty in maintaining test isolation and performance optimization.

## Decision

1. SHOULD: Core functionality tests SHOULD be grouped under tests/unit/core/ directory

## Policy Block

- SHOULD Core functionality tests SHOULD be grouped under tests/unit/core/ directory

In scope:
- All TypeScript test files (.test.ts)
- Unit tests for individual components, modules, and agents
- Integration tests for multi-component interactions
- Edge case and error scenario tests
- Test files for core functionality and agent implementations

Out of scope:
- End-to-end (e2e) tests which may have separate directory structure
- Performance/benchmark tests which may require different organization
- Test fixtures, mocks, and helper utilities (should use separate directories like __fixtures__, __mocks__)
- Non-TypeScript test files in polyglot projects

Exceptions:
- EXC-001: Legacy test files exist in non-standard locations during migration period
- EXC-002: Framework-specific tests require co-location with source code

## Rationale

- The detected pattern shows 30 files with 91.85% confidence following this structure, indicating strong organic adoption and proven effectiveness
- Separating unit and integration tests enables different CI/CD execution strategies (fast unit tests in pre-commit, slower integration tests in CI pipeline)
- Mirroring source code structure in test organization improves discoverability and makes it clear which code is covered by which tests
- Consistent naming conventions (.test.ts) and directory structure reduce cognitive load for developers and enable automated test discovery by test runners

## Consequences

Positive:
- Clear separation enables optimized CI/CD pipelines with fast feedback loops for unit tests and comprehensive validation via integration tests
- Improved test discoverability and maintainability through predictable file locations
- Easier onboarding for new developers who can quickly understand test organization patterns
- Simplified test runner configuration with glob patterns targeting specific test types

Negative:
- Requires discipline to maintain structure as codebase grows and new test types emerge
- May require refactoring existing tests that don't follow the pattern
- Edge cases about test categorization (unit vs integration) may require team discussion and documentation
- Additional directory nesting can make file paths longer

## Alternatives

- Co-locate tests with source code (e.g., Component.ts and Component.test.ts in same directory) (rejected)
  Rejected because: Makes it harder to separate test dependencies from production dependencies and complicates build/deployment processes that need to exclude tests
  When valid: May be valid for component libraries where tests serve as usage examples
- Flat test directory structure without unit/integration separation (rejected)
  Rejected because: Prevents optimized CI/CD execution strategies and makes it difficult to run only fast tests during development
  When valid: May be acceptable for very small projects with fewer than 10 test files
- Use __tests__ directory convention (common in Jest projects) (rejected)
  Rejected because: The detected pattern shows strong adoption of tests/ directory, and __tests__ convention doesn't provide clear separation between test types
  When valid: Could be considered if migrating from a Jest-heavy ecosystem

## Risks

- Inconsistent application of unit vs integration test categorization leading to misplaced test files
  Mitigation: Document clear criteria for unit vs integration tests in testing guidelines; provide examples during code review
  Owner: Engineering team leads
- Legacy tests in non-standard locations may be overlooked during test runs
  Mitigation: Audit existing test files and create migration plan; update CI configuration to explicitly include all test locations during transition
  Owner: DevOps team
- New test types (e.g., contract tests, visual regression tests) may not fit cleanly into unit/integration categories
  Mitigation: Establish process for proposing new test directory categories through ADR process; maintain flexibility for tests/ root-level organization
  Owner: Architecture team

## Implementation Notes

- Configure test runner (Jest, Vitest, etc.) with separate glob patterns for unit tests (tests/unit/**/*.test.ts) and integration tests (tests/integration/**/*.test.ts)
- Update CI/CD pipeline to run unit tests on every commit/PR and integration tests on merge to main or on schedule
- Create tests/unit/ and tests/integration/ directories if they don't exist; add README.md files explaining the purpose of each
- Establish linting rules or pre-commit hooks to validate test file locations match the expected pattern
- Document clear definitions: unit tests test single components in isolation with mocked dependencies; integration tests test multiple components working together with real or test implementations

## Continuation Context


Verify commands:
- find tests/ -name '*.test.ts' -type f | grep -E '^tests/(unit|integration)/' | wc -l
- test -d tests/unit && test -d tests/integration && echo 'Test directories exist' || echo 'Missing test directories'
- find tests/unit/agents -name '*.test.ts' 2>/dev/null | head -5

Accept when:
- All test files with .test.ts extension are located within tests/ directory
- Unit tests are organized under tests/unit/ and integration tests under tests/integration/
- Agent-specific tests exist in tests/unit/agents/ directory structure
- CI/CD pipeline configuration references the standardized test directory structure

## Enforcement

- Verified by: Automated CI checks validating test file locations match expected patterns
- Verified by: Code review checklist includes verification of test file placement
- Verified by: Pre-commit hooks or linting rules checking test file paths
- Violation handling: CI pipeline fails if tests are found outside approved directory structure
- Violation handling: Code review requires changes before approval if tests are misplaced
- Violation handling: Automated PR comments flag non-compliant test file locations
- Exception process: Developer creates issue documenting why exception is needed
- Exception process: Tech lead or architect reviews and approves exception with documented rationale
- Exception process: Exception is time-bound with migration plan if temporary
- Exception process: Exception is documented in project's testing guidelines