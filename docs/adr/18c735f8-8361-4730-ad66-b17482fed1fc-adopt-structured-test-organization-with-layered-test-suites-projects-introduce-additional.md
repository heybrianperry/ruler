# Adopt Structured Test Organization with Layered Test Suites: Projects Introduce Additional

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all test development and CI/CD pipeline configuration.

## Context

- The codebase contains 121 test files organized into distinct layers (unit, integration, e2e) with a consistent naming convention using .test.ts extensions
- Test files are systematically distributed across multiple testing concerns including agent behavior, configuration loading, CLI operations, and MCP protocol handling
- The testing infrastructure supports multiple test granularities from isolated unit tests to full end-to-end scenarios, indicating a mature testing strategy
- Evidence shows comprehensive test coverage across core functionality including agents (FirebenderAgent, ClaudeAgent, KiloCodeAgent), configuration systems, and integration points
- The pattern demonstrates a deliberate architectural choice to separate test concerns by scope and execution characteristics

## Problem Statement

Without a standardized approach to test organization and layering, test suites become difficult to maintain, slow to execute, and challenging to run selectively in CI/CD pipelines. Teams need clear guidance on where to place tests, how to structure test hierarchies, and how to ensure tests can be executed efficiently at different stages of the development and deployment lifecycle.

## Decision

1. MAY: Projects MAY introduce additional test layers (e.g., contract, performance, security) as needed for specific domains

## Policy Block

- MAY Projects MAY introduce additional test layers (e.g., contract, performance, security) as needed for specific domains

In scope:
- All automated test code in the repository
- Test execution configuration in CI/CD pipelines
- Test runner configuration files (jest.config.js, vitest.config.ts, etc.)
- Developer testing workflows and practices
- Quality gates and merge requirements

Out of scope:
- Manual testing procedures and exploratory testing
- Production monitoring and observability
- Third-party testing tools and services configuration
- Performance benchmarking infrastructure
- Security scanning and penetration testing

Exceptions:
- EX-001: Legacy test files exist in a flat structure during migration period
- EX-002: Proof-of-concept or spike work requires rapid prototyping without full test coverage

## Rationale

- The pattern appears in 121 files with 91.92% confidence, indicating this is a well-established and consistently applied practice across the codebase
- Layered test organization enables selective test execution in CI/CD pipelines, reducing feedback time for developers while maintaining comprehensive coverage
- Clear separation of test concerns improves maintainability by making it obvious where new tests should be placed and what dependencies are appropriate
- The evidence shows successful application across diverse testing scenarios (agents, configuration, CLI, protocols), demonstrating the pattern's versatility and robustness

## Consequences

Positive:
- Faster CI/CD feedback loops by running quick unit tests first and expensive e2e tests last
- Improved test maintainability through clear organizational boundaries and consistent naming
- Better developer experience with ability to run specific test layers during development
- Reduced test flakiness by enforcing appropriate isolation levels for each test type
- Easier onboarding for new team members with self-documenting test structure

Negative:
- Initial setup cost to establish test infrastructure and directory structure
- Requires discipline to place tests in the correct layer, with potential for misclassification
- May lead to test duplication if boundaries between layers are not clearly understood
- Additional complexity in test runner configuration to support multiple test layers
- Potential for slower overall test suite execution if layers are not properly parallelized

## Alternatives

- Flat test directory structure with all tests in a single tests/ folder (rejected)
  Rejected because: Does not support selective test execution, makes it difficult to identify test scope, and leads to slower CI/CD pipelines as all tests must run together
  When valid: Only appropriate for very small projects with fewer than 20 test files
- Co-locate tests with source code (e.g., Component.ts and Component.test.ts in same directory) (rejected)
  Rejected because: While this improves discoverability, it does not solve the layering problem and makes it harder to run tests by scope in CI/CD pipelines
  When valid: Can be combined with this ADR by using subdirectories like __tests__/unit/ within source directories
- Tag-based test organization using test framework annotations instead of directory structure (deferred)
  Rejected because: Requires more sophisticated test runner configuration and is less discoverable than directory structure
  When valid: May be adopted as a complementary approach for cross-cutting concerns like smoke tests or regression suites

## Risks

- Developers may misclassify tests, placing integration tests in unit/ or vice versa, undermining the benefits of layering
  Mitigation: Provide clear documentation with examples, implement automated checks for common anti-patterns (e.g., network calls in unit tests), and include test classification in code review checklist
  Owner: Engineering team leads
- Test execution time may increase if layers are not properly configured for parallel execution
  Mitigation: Configure CI/CD to run test layers in parallel where possible, monitor test execution times, and optimize slow tests
  Owner: DevOps/Platform team
- Rigid layer boundaries may discourage writing tests if developers are unsure where a test belongs
  Mitigation: Establish clear guidelines with decision tree, provide examples of edge cases, and encourage discussion in code reviews rather than blocking contributions
  Owner: Engineering team

## Implementation Notes

- Start by creating the three core test directories (tests/unit/, tests/integration/, tests/e2e/) and configure test runner to recognize each layer
- Update CI/CD pipeline configuration to run test layers sequentially with separate reporting for each layer
- Document clear criteria for each test layer with examples: unit (no I/O, fast, isolated), integration (controlled dependencies, moderate speed), e2e (full system, slower)
- Consider using test framework features like jest.config.js projects or vitest workspace to configure different settings per layer (timeouts, setup files, etc.)
- Migrate existing tests gradually, starting with clearly identifiable unit tests, and use the migration as a learning opportunity to refine layer definitions

## Continuation Context


Verify commands:
- find tests/ -type f -name '*.test.ts' | grep -E '^tests/(unit|integration|e2e)/' | wc -l
- grep -r 'describe\|test\|it' tests/unit/ | grep -E '(fetch|http|database|fs\.write)' && echo 'Found I/O in unit tests' || echo 'Unit tests clean'
- test -d tests/unit && test -d tests/integration && test -d tests/e2e && echo 'Test directories exist' || echo 'Missing test directories'

Accept when:
- All test files are located within tests/unit/, tests/integration/, or tests/e2e/ directories
- Unit tests contain no external I/O operations (network, database, file system writes)
- CI/CD pipeline configuration includes separate stages or jobs for each test layer
- Test execution reports clearly identify which layer each test belongs to

## Enforcement

- Verified by: Automated CI/CD pipeline checks verify test directory structure
- Verified by: Code review checklist includes test layer classification verification
- Verified by: Static analysis tools scan for I/O operations in unit test directories
- Verified by: Pre-commit hooks validate test file naming conventions
- Violation handling: CI build fails if tests are found outside approved directory structure
- Violation handling: Code review requires changes if tests are misclassified based on their dependencies
- Violation handling: Automated comments on pull requests flag potential test layer violations
- Violation handling: Quarterly audit of test organization with remediation plan for violations
- Exception process: Developer documents exception rationale in test file comments
- Exception process: Tech lead reviews and approves exception during code review
- Exception process: Exception is logged in technical debt tracking system with remediation plan
- Exception process: Exceptions are reviewed quarterly to determine if pattern needs adjustment