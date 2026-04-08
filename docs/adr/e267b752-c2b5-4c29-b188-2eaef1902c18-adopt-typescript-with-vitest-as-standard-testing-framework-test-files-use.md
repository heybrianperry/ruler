# Adopt TypeScript with Vitest as Standard Testing Framework: Test Files Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains 14 test files using TypeScript with .test.ts extension, indicating a standardized testing approach across unit, integration, and e2e test layers
- Test files are organized by test type (unit, integration, e2e) and cover core functionality including CLI, MCP, agents, and file system utilities
- The testing framework supports multiple test categories including CLI tests, unit tests for core components, integration tests for nested configurations and skills, and end-to-end tests
- Pattern detected with 91.86% confidence across diverse test scenarios suggests deliberate architectural choice for test infrastructure
- Testing infrastructure is integrated into CI/CD pipeline as evidenced by consistent test file naming and organization patterns

## Problem Statement

The project requires a consistent, type-safe testing framework that integrates seamlessly with TypeScript codebases, supports multiple test types (unit, integration, e2e), and provides fast feedback loops in CI/CD pipelines while maintaining developer productivity and code quality standards.

## Decision

1. MUST: All test files MUST use TypeScript with the .test.ts file extension

## Policy Block

- MUST All test files MUST use TypeScript with the .test.ts file extension

In scope:
- All TypeScript source code requiring automated testing
- Unit tests for core utilities, agents, and MCP components
- Integration tests for configuration handling and skills isolation
- End-to-end tests for CLI and ruler initialization
- CI/CD pipeline test execution stages

Out of scope:
- Manual testing procedures and QA workflows
- Performance benchmarking tests (may use different tooling)
- Load testing and stress testing scenarios
- Third-party library tests (covered by upstream projects)

## Rationale

- TypeScript testing provides type safety and catches errors at compile time, reducing runtime failures and improving code quality
- Vitest offers native TypeScript support, fast execution with ESM compatibility, and excellent developer experience with watch mode and hot module replacement
- Organized test structure (unit/integration/e2e) enables clear separation of concerns and appropriate testing strategies for different system layers
- Pattern detected across 14 files with 91.86% confidence indicates this is an established, working practice that has proven effective for the project's needs

## Consequences

Positive:
- Type-safe tests catch interface mismatches and type errors before runtime, improving reliability
- Fast test execution with Vitest enables rapid feedback loops during development
- Clear test organization makes it easy for developers to locate and write appropriate tests
- Consistent testing patterns across the codebase reduce cognitive load and onboarding time for new contributors
- Native TypeScript support eliminates transpilation complexity and configuration overhead

Negative:
- TypeScript compilation adds slight overhead to test execution time compared to plain JavaScript
- Developers must maintain type definitions for test fixtures and mocks
- Learning curve for team members unfamiliar with TypeScript or Vitest
- Potential for over-reliance on type checking leading to insufficient runtime validation in tests

## Alternatives

- Use Jest with TypeScript via ts-jest (rejected)
  Rejected because: Jest requires additional transpilation layer (ts-jest) which adds complexity and slower execution compared to Vitest's native ESM and TypeScript support
  When valid: For projects already heavily invested in Jest ecosystem or requiring specific Jest plugins not available in Vitest
- Use plain JavaScript with JSDoc type annotations (rejected)
  Rejected because: JSDoc provides weaker type checking and less IDE support compared to native TypeScript, reducing developer productivity and type safety guarantees
  When valid: For projects with strict no-build-step requirements or teams without TypeScript expertise
- Use Mocha/Chai with TypeScript (rejected)
  Rejected because: Requires separate assertion library and more configuration, lacks modern features like built-in mocking and watch mode that Vitest provides out of the box
  When valid: For legacy projects already using Mocha/Chai or requiring specific Mocha reporters

## Risks

- Vitest is relatively newer than Jest and may have less community support or undiscovered edge cases
  Mitigation: Monitor Vitest issue tracker, maintain fallback plan to migrate to Jest if critical issues arise, contribute fixes upstream when possible
  Owner: Engineering Team
- TypeScript version upgrades may introduce breaking changes in test type definitions
  Mitigation: Pin TypeScript versions in package.json, test upgrades in isolated branches, maintain comprehensive test coverage to catch regressions
  Owner: Engineering Team
- Test execution time may grow as test suite expands, slowing CI/CD pipelines
  Mitigation: Implement test parallelization, use Vitest's workspace feature for test sharding, monitor test performance metrics and optimize slow tests
  Owner: DevOps Team

## Implementation Notes

- Configure Vitest in vitest.config.ts with appropriate test patterns matching tests/**/*.test.ts
- Set up separate test configurations for unit, integration, and e2e tests if they require different environments or timeouts
- Use Vitest's workspace feature to organize tests by type and enable parallel execution across test categories
- Establish naming conventions for test files that clearly indicate the component under test and test type
- Configure TypeScript compiler options for tests to enable strict type checking while allowing test-specific flexibility

## Continuation Context


Verify commands:
- find tests -name '*.test.ts' -type f | wc -l | grep -E '^[1-9][0-9]*$'
- grep -r "import.*vitest" tests/ --include='*.test.ts' | wc -l
- test -d tests/unit && test -d tests/integration && test -d tests/e2e && echo 'Test structure valid'

Accept when:
- All test files use .test.ts extension and are written in TypeScript
- Test files are organized into unit/, integration/, and e2e/ directories based on test type
- Vitest is configured and all tests execute successfully in CI/CD pipeline
- Test coverage reports are generated and meet minimum threshold requirements

## Enforcement

- Verified by: CI/CD pipeline checks for test file naming conventions using lint rules
- Verified by: Pre-commit hooks validate test file extensions and directory structure
- Verified by: Code review process verifies new tests follow organizational patterns
- Verified by: Automated test execution in CI fails if tests are not properly structured or fail to run
- Violation handling: CI pipeline fails if test files do not match .test.ts pattern
- Violation handling: Pull requests are blocked if tests are not in appropriate directories
- Violation handling: Linting errors are raised for improperly named or located test files
- Violation handling: Code review feedback requests corrections before merge approval
- Exception process: Document exception rationale in ADR amendment or technical debt ticket
- Exception process: Obtain approval from tech lead or architecture review board
- Exception process: Add exception to .eslintignore or test configuration with explanatory comment
- Exception process: Schedule remediation work if exception is temporary