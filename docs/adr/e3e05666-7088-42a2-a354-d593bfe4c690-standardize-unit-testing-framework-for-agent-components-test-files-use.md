# Standardize Unit Testing Framework for Agent Components: Test Files Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent component development and testing activities within the codebase.

## Context

- The codebase contains multiple agent implementations (AmpAgent, WarpAgent, JulesAgent, WindsurfAgent, OpenCodeAgent, JunieAgent) that require consistent testing approaches
- Unit tests have been established across 8 files with a consistent pattern, indicating a deliberate architectural choice for developer tooling
- Testing infrastructure needs to support both valid agent implementations and invalid agent scenarios to ensure robust error handling
- CLI handlers require unit testing to validate command-line interface behavior and integration with agent components
- A standardized testing framework enables faster onboarding, consistent quality assurance, and maintainable test suites across the agent ecosystem

## Problem Statement

Without a standardized unit testing framework for agent components, the codebase risks inconsistent test coverage, duplicated testing logic, difficulty in maintaining tests across multiple agent implementations, and challenges in ensuring uniform quality standards. The presence of 8 test files with high significance (0.90-0.93) indicates an established pattern that needs formal documentation to prevent drift and ensure future agent implementations follow the same testing standards.

## Decision

1. MUST: Test files MUST use the `.test.ts` extension to be recognized by the testing framework

## Policy Block

- MUST Test files MUST use the `.test.ts` extension to be recognized by the testing framework

In scope:
- All agent component implementations (AmpAgent, WarpAgent, JulesAgent, WindsurfAgent, OpenCodeAgent, JunieAgent, and future agents)
- CLI handler components and command-line interface logic
- Error handling and invalid input scenarios
- Unit-level testing of isolated components

Out of scope:
- Integration tests that span multiple components or systems
- End-to-end tests that require full system deployment
- Performance or load testing
- Manual testing procedures

Exceptions:
- EXC-001: Legacy agent implementations that predate this ADR and are scheduled for deprecation within 6 months
- EXC-002: Prototype or experimental agents in research/spike branches not intended for production

## Rationale

- Pattern detected across 8 files with 92.19% confidence indicates this is an established, intentional architectural decision rather than coincidental similarity
- Consistent test file organization and naming conventions reduce cognitive load for developers and enable automated tooling to discover and execute tests reliably
- Separating unit tests by component type (agents vs. cli) provides clear boundaries and makes the test suite more navigable as the codebase scales
- Explicit testing of invalid scenarios demonstrates a mature approach to quality assurance that prevents regressions in error handling paths

## Consequences

Positive:
- Developers can quickly locate and understand test coverage for any agent implementation through consistent file naming and organization
- New agent implementations benefit from clear examples and patterns established by existing test suites
- Automated test discovery and execution is simplified through standardized file extensions and directory structure
- Code review efficiency improves as reviewers know exactly where to find tests and what patterns to expect
- Refactoring confidence increases with comprehensive unit test coverage across all agent components

Negative:
- Rigid directory structure may require migration effort if testing framework or organizational needs change significantly
- Developers must learn and follow the established conventions, adding a small learning curve for new team members
- Test file proliferation in the tests/unit directory may require additional organization as the number of agents grows beyond 20-30 implementations
- Maintaining consistency across all test files requires ongoing code review vigilance and potentially linting rules

## Alternatives

- Co-locate tests with source files (e.g., `src/agents/AmpAgent.test.ts` alongside `src/agents/AmpAgent.ts`) (rejected)
  Rejected because: Separating tests into a dedicated `tests/` directory provides clearer separation of concerns, simplifies build configurations that exclude tests from production bundles, and aligns with common TypeScript/Node.js project conventions
  When valid: May be reconsidered for very small utility modules or if the team adopts a component-based architecture where each component is fully self-contained
- Use a flat test directory structure without subdirectories (all tests in `tests/` root) (rejected)
  Rejected because: As the codebase scales with multiple agent types and CLI components, a flat structure would become unmanageable and make it difficult to locate relevant tests quickly
  When valid: Only appropriate for very small projects with fewer than 10 test files total
- Adopt behavior-driven development (BDD) style test organization with feature-based directories (deferred)
  Rejected because: Current unit test approach is working well with high confidence (92.19%), but BDD could be valuable for higher-level integration or acceptance tests
  When valid: Should be reconsidered when adding integration test suites or when stakeholders require executable specifications

## Risks

- Test suite execution time may grow linearly with the number of agents, potentially slowing down CI/CD pipelines
  Mitigation: Implement parallel test execution, monitor test performance metrics, and establish test execution time budgets per file (e.g., <5 seconds per unit test file)
  Owner: Engineering Team / DevOps
- Inconsistent test quality or coverage despite standardized structure if developers write minimal or superficial tests
  Mitigation: Enforce code coverage thresholds (e.g., 80% minimum), implement test quality checks in code review, and provide test templates or generators for new agents
  Owner: Engineering Team / Tech Leads
- Testing framework lock-in may make it difficult to migrate to alternative testing tools if requirements change
  Mitigation: Abstract test utilities and helpers into reusable modules, document testing framework dependencies clearly, and periodically evaluate alternative frameworks during architecture reviews
  Owner: Architecture Team

## Implementation Notes

- Create a test template or generator script that scaffolds new agent test files with the correct naming convention and boilerplate structure
- Configure the testing framework (likely Jest or Vitest based on .test.ts extension) to automatically discover tests in the tests/unit directory tree
- Establish code coverage reporting that maps coverage back to source files and enforces minimum thresholds in CI
- Document testing patterns and best practices in a TESTING.md guide with examples from existing agent tests
- Set up pre-commit hooks or linting rules to validate test file naming conventions and directory structure

## Continuation Context


Verify commands:
- find tests/unit -name '*.test.ts' | grep -E 'tests/unit/(agents|cli)/.+\.test\.ts$'
- npm test -- --coverage --coverageThreshold='{"global":{"lines":80}}'
- ls tests/unit/agents/*.test.ts | wc -l | grep -E '^[6-9]|^[1-9][0-9]+$'

Accept when:
- All test files follow the naming pattern `*.test.ts` and are located in appropriate subdirectories under `tests/unit/`
- Test suite executes successfully with at least 80% code coverage across agent components
- At least 6 agent test files exist in `tests/unit/agents/` directory, maintaining the established pattern

## Enforcement

- Verified by: Automated CI pipeline checks that run on every pull request to verify test file naming and location
- Verified by: Code coverage reports generated during CI builds with enforced minimum thresholds
- Verified by: Code review checklist items requiring reviewers to verify test presence and quality
- Verified by: Static analysis tools or custom linting rules that validate test file structure
- Violation handling: Pull requests that add new agents without corresponding unit tests in the correct location will fail CI checks and be blocked from merging
- Violation handling: Coverage drops below threshold will fail the build and require additional tests before merge approval
- Violation handling: Code reviewers will request changes for any test files that don't follow the established naming and organizational conventions
- Violation handling: Quarterly architecture reviews will audit test suite compliance and identify technical debt for remediation
- Exception process: Developer submits exception request via GitHub issue or architecture decision log with justification
- Exception process: Engineering lead or architect reviews the request within 2 business days
- Exception process: If approved, exception is documented in the component's README or inline comments with expiration date
- Exception process: Exceptions are tracked in a central registry and reviewed quarterly for closure or extension