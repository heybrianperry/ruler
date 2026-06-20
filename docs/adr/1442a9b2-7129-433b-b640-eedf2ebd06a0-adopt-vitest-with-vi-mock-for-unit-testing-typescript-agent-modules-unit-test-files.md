# Adopt Vitest with vi.mock() for Unit Testing TypeScript Agent Modules: Unit Test Files

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all unit test development in the codebase.

## Context

- The codebase contains multiple agent implementations (AmpAgent, WarpAgent, JulesAgent, WindsurfAgent, OpenCodeAgent, JunieAgent) that require isolated unit testing without external dependencies
- Unit tests need to verify agent behavior and CLI handler logic in isolation, requiring a mocking framework that supports TypeScript and ES modules
- The testing.mocking facet pattern was detected across 8 test files with 92.19% confidence, indicating a consistent testing approach
- Modern TypeScript projects require fast test execution and native ESM support, which traditional testing frameworks like Jest struggle to provide efficiently
- The pattern shows consistent use of Vitest's vi.mock() API for module mocking, suggesting a deliberate architectural choice for the testing infrastructure

## Problem Statement

How should the codebase standardize unit testing for TypeScript agent modules and CLI handlers to ensure fast, reliable, isolated tests with proper mocking capabilities while maintaining compatibility with modern ES module syntax?

## Decision

1. MUST: Unit test files MUST follow the naming convention *.test.ts and be located in the tests/unit directory structure

## Policy Block

- MUST Unit test files MUST follow the naming convention *.test.ts and be located in the tests/unit directory structure

In scope:
- All TypeScript unit tests in the tests/unit directory
- Agent module tests (AmpAgent, WarpAgent, JulesAgent, WindsurfAgent, OpenCodeAgent, JunieAgent)
- CLI handler tests
- Any new unit tests added to the codebase

Out of scope:
- Integration tests that intentionally test real dependencies
- End-to-end tests
- Performance benchmarking tests
- Tests written in languages other than TypeScript

Exceptions:
- EX-001: Legacy tests exist that use Jest and migration would require significant refactoring
- EX-002: Testing framework-specific behavior that requires a different testing tool

## Rationale

- Vitest provides native ESM support and TypeScript compatibility out of the box, eliminating configuration complexity and improving developer experience
- The vi.mock() API offers a clean, intuitive interface for module mocking that works seamlessly with TypeScript's type system
- Pattern detection across 8 files with 92.19% confidence indicates this approach is already successfully adopted and proven in the codebase
- Vitest's performance characteristics (fast startup, parallel execution) significantly improve test feedback loops compared to traditional frameworks

## Consequences

Positive:
- Fast test execution with minimal configuration overhead enables rapid development cycles
- Consistent mocking patterns across all agent tests improve code maintainability and reduce cognitive load
- Native TypeScript and ESM support eliminates common configuration issues and transpilation complexity
- Strong type safety in tests catches errors at compile time rather than runtime

Negative:
- Team members unfamiliar with Vitest will need to learn new API patterns (though similar to Jest)
- Existing Jest-based tests would require migration effort if present in the codebase
- Vitest ecosystem is younger than Jest, potentially having fewer third-party integrations
- Mock setup at module level can make tests harder to understand for developers expecting inline mocking

## Alternatives

- Use Jest with ts-jest for TypeScript unit testing (rejected)
  Rejected because: Jest requires additional configuration for ESM support, has slower startup times, and the codebase has already standardized on Vitest as evidenced by the pattern detection
  When valid: For projects with existing Jest infrastructure and no ESM requirements
- Use native Node.js test runner with manual mocking (rejected)
  Rejected because: Lacks mature mocking utilities, requires more boilerplate code, and provides less developer-friendly assertion APIs
  When valid: For minimal dependency projects or when testing Node.js-specific features
- Use Mocha with Sinon for mocking (rejected)
  Rejected because: Requires multiple libraries for complete testing solution, less integrated TypeScript support, and steeper learning curve for modern ESM patterns
  When valid: For legacy codebases already using Mocha/Sinon stack

## Risks

- Over-mocking can lead to tests that pass but don't reflect real-world behavior, creating false confidence
  Mitigation: Complement unit tests with integration tests that use real dependencies; establish code review guidelines for appropriate mocking boundaries
  Owner: Engineering team and QA leads
- Vitest API changes in future versions could require test refactoring
  Mitigation: Pin Vitest version in package.json; monitor release notes; allocate time for gradual upgrades with deprecation warnings
  Owner: DevOps and engineering team
- Module-level mocks can create hidden dependencies between tests if not properly isolated
  Mitigation: Enforce beforeEach/afterEach hooks for mock cleanup; use vi.clearAllMocks() or vi.resetAllMocks() consistently; include in code review checklist
  Owner: Engineering team

## Implementation Notes

- Install Vitest as a dev dependency: npm install -D vitest or yarn add -D vitest
- Configure vitest.config.ts with appropriate test file patterns matching tests/unit/**/*.test.ts
- Use vi.mock('module-path') at the top of test files before imports to ensure proper hoisting
- Implement beforeEach(() => vi.clearAllMocks()) in test suites to prevent mock state leakage between tests
- For agent tests, mock external dependencies like file system, network calls, and third-party libraries consistently
- Add npm scripts for running tests: 'test:unit': 'vitest run' and 'test:unit:watch': 'vitest watch'

## Continuation Context


Verify commands:
- grep -r "vi.mock" tests/unit/ | wc -l
- grep -r "from 'vitest'" tests/unit/**/*.test.ts | wc -l
- find tests/unit -name '*.test.ts' -type f | wc -l
- grep -r "describe\|test\|it" tests/unit/ | head -5

Accept when:
- All unit test files in tests/unit/ use Vitest imports (import { describe, test, expect, vi } from 'vitest')
- At least 80% of agent test files use vi.mock() for dependency isolation
- Test files follow naming convention *.test.ts and are located in tests/unit directory structure
- CI pipeline successfully runs vitest command and all unit tests pass

## Enforcement

- Verified by: Automated CI pipeline checks that run vitest and fail on test failures
- Verified by: Code review checklist requiring Vitest usage for new unit tests
- Verified by: Static analysis tools checking for consistent test file naming patterns
- Verified by: Pre-commit hooks validating test file structure and imports
- Violation handling: Pull requests introducing non-Vitest unit tests are blocked in code review
- Violation handling: CI pipeline fails if test files don't match expected patterns or use incorrect frameworks
- Violation handling: Automated comments on PRs flag violations with links to this ADR
- Violation handling: Quarterly audits identify and prioritize migration of non-compliant tests
- Exception process: Developer submits exception request via GitHub issue with justification
- Exception process: Tech lead reviews technical merit and approves/rejects within 2 business days
- Exception process: Approved exceptions are documented in test file headers with EX-XXX reference
- Exception process: Exception registry is maintained and reviewed quarterly for potential pattern changes