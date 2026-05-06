# Adopt TypeScript with Jest as Standard Testing Framework: Test Files Organized

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all test file creation and test framework configuration decisions.

## Context

- The codebase contains 30 test files following a consistent pattern of TypeScript-based testing with Jest framework, indicating a deliberate architectural choice
- Test files are organized into unit and integration test directories (tests/unit/*, tests/integration/*), demonstrating a structured testing strategy
- Multiple agent implementations (ClaudeAgent, RooCodeAgent, OpenHandsAgent, KiroAgent, QwenCodeAgent, TraeAgent, MistralVibeAgent) all use the same testing approach, suggesting this is a cross-cutting architectural standard
- The pattern shows 91.85% confidence across 30 files, indicating strong consistency and widespread adoption throughout the codebase
- Test files use .test.ts extension convention, enabling automated test discovery and execution in CI/CD pipelines

## Problem Statement

Without a standardized testing framework and language choice, teams may adopt inconsistent testing approaches leading to fragmented tooling, incompatible test suites, increased maintenance burden, and difficulty in establishing unified CI/CD pipelines. The codebase needs a clear decision on testing technology to ensure consistency, maintainability, and effective quality assurance practices.

## Decision

1. MUST: Test files MUST be organized into tests/unit/ and tests/integration/ directories based on test scope

## Policy Block

- MUST Test files MUST be organized into tests/unit/ and tests/integration/ directories based on test scope

In scope:
- All TypeScript source code requiring automated testing
- Unit tests for individual modules, classes, and functions
- Integration tests for component interactions and workflows
- Agent implementation testing across all agent types
- Core engine and utility function testing
- Configuration and skills testing

Out of scope:
- End-to-end tests that may require different frameworks (e.g., Playwright, Cypress)
- Performance and load testing which may use specialized tools
- Manual testing and exploratory testing activities
- Third-party library testing (responsibility of library maintainers)
- Documentation examples that are not executable tests

Exceptions:
- EXC-001: Legacy JavaScript test files exist and require maintenance without full migration budget
- EXC-002: Specialized testing scenarios require framework-specific features not available in Jest

## Rationale

- The pattern detection identified 30 test files with 91.85% confidence, demonstrating strong existing adoption and consistency across the codebase
- TypeScript provides type safety and better IDE support for test development, reducing runtime errors and improving developer productivity
- Jest is a mature, widely-adopted testing framework with excellent TypeScript support, comprehensive features (mocking, coverage, snapshots), and active community maintenance
- Standardizing on a single testing framework reduces cognitive load, simplifies CI/CD configuration, and enables shared testing utilities and patterns across the codebase

## Consequences

Positive:
- Consistent testing approach across all modules improves code maintainability and reduces onboarding time for new developers
- TypeScript type checking in tests catches errors earlier and provides better refactoring support
- Jest's built-in features (mocking, coverage, watch mode) eliminate need for multiple testing dependencies
- Unified test execution in CI/CD pipelines simplifies build configuration and reduces pipeline complexity
- Strong IDE integration provides better test development experience with autocomplete and inline error detection

Negative:
- Teams familiar with alternative frameworks (Mocha, Vitest, AVA) face learning curve and migration effort
- Jest can be slower than some newer alternatives for large test suites, potentially increasing CI/CD execution time
- TypeScript compilation adds build step overhead compared to pure JavaScript testing
- Framework lock-in makes future migration to alternative testing tools more costly
- Some edge cases may require workarounds or additional configuration specific to Jest's execution model

## Alternatives

- Use Vitest as the primary testing framework with TypeScript (rejected)
  Rejected because: While Vitest offers faster execution and better Vite integration, the existing codebase has already standardized on Jest with 30 test files. Migration cost outweighs performance benefits, and Jest's maturity and ecosystem provide better long-term stability.
  When valid: Consider for new greenfield projects or when migrating to Vite-based build system
- Use Mocha with Chai for testing with TypeScript (rejected)
  Rejected because: Mocha requires additional libraries for assertions, mocking, and coverage, increasing dependency complexity. Jest provides an all-in-one solution with better developer experience and simpler configuration.
  When valid: Only if specific Mocha plugins are required for specialized testing scenarios
- Allow multiple testing frameworks based on team preference (rejected)
  Rejected because: Multiple frameworks fragment the codebase, increase maintenance burden, complicate CI/CD pipelines, and create inconsistent testing patterns. Standardization provides better long-term maintainability.
  When valid: Never recommended for production codebases requiring consistency

## Risks

- Jest performance degradation as test suite grows, leading to slow CI/CD pipelines
  Mitigation: Implement test parallelization, use Jest's --maxWorkers configuration, monitor test execution times, and optimize slow tests. Consider test sharding for large suites.
  Owner: Engineering team and DevOps
- TypeScript version incompatibilities or breaking changes affecting test compilation
  Mitigation: Pin TypeScript and Jest versions in package.json, maintain comprehensive upgrade testing process, and monitor Jest and TypeScript release notes for breaking changes.
  Owner: Engineering team
- Developers bypassing testing standards due to perceived complexity or time pressure
  Mitigation: Provide test templates and examples, integrate automated checks in CI/CD to enforce test presence, conduct code reviews focusing on test quality, and provide training on Jest best practices.
  Owner: Tech leads and engineering managers

## Implementation Notes

- Configure Jest with TypeScript support using ts-jest or @swc/jest for faster compilation
- Create shared test utilities and helper functions in tests/helpers/ or tests/utils/ directories to promote code reuse
- Set up Jest configuration file (jest.config.js or jest.config.ts) with appropriate test patterns, coverage thresholds, and module resolution
- Integrate Jest into package.json scripts (test, test:unit, test:integration, test:watch, test:coverage) for consistent execution
- Configure IDE plugins (Jest Runner for VS Code) to enable inline test execution and debugging
- Establish naming conventions and test structure patterns (Arrange-Act-Assert, Given-When-Then) in testing guidelines documentation

## Continuation Context


Verify commands:
- find tests -name '*.test.ts' | wc -l | grep -v '^0$'
- grep -r "from '@jest'" tests/ || grep -r "from 'jest'" tests/ || grep -r 'describe\|test\|it\|expect' tests/
- test -f jest.config.js || test -f jest.config.ts || grep -q '"jest"' package.json
- npm test 2>&1 | grep -E '(Tests:|Test Suites:)' || yarn test 2>&1 | grep -E '(Tests:|Test Suites:)'

Accept when:
- At least one .test.ts file exists in the tests/ directory structure
- Jest is configured in the project (jest.config file exists or jest configuration in package.json)
- Test execution command (npm test or yarn test) successfully runs Jest and reports test results
- CI/CD pipeline includes Jest test execution step that gates deployments

## Enforcement

- Verified by: Automated CI/CD pipeline checks that execute Jest tests on every commit and pull request
- Verified by: Code review process verifying that new features include corresponding .test.ts files
- Verified by: Static analysis tools checking for .test.ts file presence alongside source files
- Verified by: Pre-commit hooks running Jest tests to catch failures before code is pushed
- Violation handling: CI/CD pipeline fails if Jest tests do not pass, blocking merge and deployment
- Violation handling: Pull requests without tests for new functionality are flagged in code review and require justification or test addition
- Violation handling: Coverage reports are generated and reviewed to identify untested code paths
- Violation handling: Periodic audits identify modules lacking adequate test coverage for remediation
- Exception process: Developer documents exception rationale in pull request description or ADR
- Exception process: Tech lead or architect reviews and approves exception based on valid technical constraints
- Exception process: Exception is time-bound with documented plan to achieve compliance
- Exception process: Exceptions are tracked in technical debt register for future resolution