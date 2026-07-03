# Standardize on Vitest for Unit Testing Agent Modules: Shared Test Utilities

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all agent module unit tests and applies to current and future agent implementations.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, WarpAgent, WindsurfAgent, JunieAgent, JulesAgent) that require consistent unit testing approaches
- A uniform testing framework across all agent modules enables consistent test patterns, shared utilities, and predictable developer experience
- The test files are located in tests/unit/agents/ directory following a consistent naming convention (*.test.ts), indicating an established testing structure
- Pattern detection identified 5 files with 92.50% confidence and significance, demonstrating strong consistency in testing approach across agent implementations
- TypeScript-based agent modules require a testing framework with native TypeScript support and fast execution for developer productivity

## Problem Statement

Without a standardized testing framework for agent modules, the codebase risks inconsistent test patterns, duplicated testing utilities, incompatible mocking strategies, and fragmented developer knowledge. This creates maintenance overhead and reduces confidence in test coverage across different agent implementations.

## Decision

1. SHOULD: Shared test utilities and mocks for agent testing SHOULD be extracted to common test helper modules

## Policy Block

- SHOULD Shared test utilities and mocks for agent testing SHOULD be extracted to common test helper modules

In scope:
- All unit tests for agent modules in tests/unit/agents/
- New agent implementations added to the codebase
- Refactored or updated existing agent test suites
- Test utilities and helpers specifically for agent testing

Out of scope:
- Integration tests that may use different testing approaches
- End-to-end tests that exercise full system behavior
- Performance benchmarks or load tests
- Tests for non-agent modules which may have different framework requirements

Exceptions:
- EXC-001: Legacy agent tests exist in a different framework and immediate migration would block critical work

## Rationale

- Pattern detection identified 5 agent test files with 92.50% confidence, demonstrating strong existing adoption and consistency
- Vitest provides native TypeScript support, fast execution with ESM compatibility, and a Jest-compatible API that reduces learning curve
- Standardizing on a single framework reduces cognitive overhead for developers working across different agent modules
- Consistent testing patterns enable better code review, easier onboarding, and more reliable CI/CD pipelines

## Consequences

Positive:
- Developers can apply knowledge from one agent test suite to another without learning different frameworks
- Shared test utilities and mocking strategies can be reused across all agent implementations
- Faster test execution with Vitest's performance optimizations improves developer feedback loops
- Consistent test structure makes it easier to identify gaps in test coverage across agents

Negative:
- Teams already familiar with alternative frameworks (Jest, Mocha) must learn Vitest-specific features
- Existing tests in other frameworks would require migration effort if standardization is enforced retroactively
- Framework lock-in creates dependency on Vitest's continued maintenance and evolution
- Some edge cases or specific testing needs might be better served by specialized testing tools

## Alternatives

- Use Jest as the standard testing framework (rejected)
  Rejected because: Jest has slower startup times and less optimal ESM support compared to Vitest, which is critical for TypeScript module testing performance
  When valid: Valid for projects with existing large Jest test suites where migration cost outweighs performance benefits
- Allow each agent module to choose its own testing framework (rejected)
  Rejected because: Fragmented testing approaches create inconsistency, duplicate effort in test utilities, and increase maintenance burden across the codebase
  When valid: Valid only in polyglot environments where agents are implemented in different languages requiring language-specific frameworks
- Use native Node.js test runner (rejected)
  Rejected because: Native test runner lacks mature ecosystem, snapshot testing, and advanced mocking capabilities that complex agent testing requires
  When valid: Valid for simple projects with minimal testing requirements and desire to avoid external dependencies

## Risks

- Vitest framework may have breaking changes in future versions that require test suite updates
  Mitigation: Pin Vitest version in package.json, monitor release notes, and allocate time for planned upgrades with thorough testing
  Owner: Engineering team
- Developers unfamiliar with Vitest may write suboptimal tests or misuse framework features
  Mitigation: Provide Vitest training documentation, code examples, and establish test review guidelines in pull request process
  Owner: Engineering team
- Migration of existing non-Vitest tests could introduce regressions if not carefully executed
  Mitigation: Create migration guide, run both old and new tests in parallel during transition, and validate coverage metrics remain stable
  Owner: Engineering team

## Implementation Notes

- Ensure vitest is installed as a dev dependency with appropriate TypeScript configuration in vitest.config.ts
- Create shared test utilities in tests/unit/agents/helpers/ for common agent mocking and setup patterns
- Configure CI pipeline to run vitest with coverage reporting and fail builds on coverage threshold violations
- Document Vitest best practices specific to agent testing in the project's testing guide or CONTRIBUTING.md

## Continuation Context


Verify commands:
- grep -r "from 'vitest'" tests/unit/agents/*.test.ts | wc -l
- find tests/unit/agents -name '*.test.ts' -type f | wc -l
- npx vitest run tests/unit/agents/ --reporter=verbose

Accept when:
- All agent test files in tests/unit/agents/ import test functions from 'vitest' package
- Test files follow naming convention {AgentName}.test.ts and execute successfully with vitest runner
- No agent unit tests use alternative testing frameworks (jest, mocha, ava) in the tests/unit/agents/ directory

## Enforcement

- Verified by: Automated CI checks that verify all .test.ts files in tests/unit/agents/ import from vitest
- Verified by: Code review process checks for consistent test framework usage in new agent implementations
- Verified by: Pre-commit hooks that validate test file naming conventions and locations
- Violation handling: CI pipeline fails if agent tests are found using non-Vitest frameworks
- Violation handling: Pull requests with non-compliant tests are blocked until corrected
- Violation handling: Automated alerts notify team leads of framework inconsistencies detected in merged code
- Exception process: Developer submits exception request to engineering lead with technical justification
- Exception process: Exception requires documented migration plan if temporary, or architectural justification if permanent
- Exception process: Approved exceptions are recorded in ADR updates and reviewed quarterly for continued validity