# Adopt TypeScript Unit Testing Pattern for Agent Components: Each Agent Test

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent component development and testing activities.

## Context

- The codebase contains multiple agent implementations (AmpAgent, OpenCodeAgent, JulesAgent, JunieAgent, WindsurfAgent, WarpAgent) that require consistent testing approaches
- A standardized unit testing pattern has emerged across 8 test files with 92.19% confidence, indicating a deliberate architectural choice
- Agent components represent critical business logic that requires isolation testing to ensure reliability and maintainability
- The testing pattern includes both positive test cases for valid agents and negative test cases for invalid agent scenarios
- CLI handlers and agent implementations follow a consistent testing structure that enables rapid development and regression detection

## Problem Statement

Without a standardized unit testing approach for agent components, the codebase risks inconsistent test coverage, difficult-to-maintain test suites, and reduced confidence in agent behavior. The proliferation of multiple agent types necessitates a unified testing strategy that ensures all agents meet quality standards while enabling independent development and testing of each component.

## Decision

1. SHOULD: Each agent test file SHOULD cover both success paths and failure scenarios for comprehensive coverage

## Policy Block

- SHOULD Each agent test file SHOULD cover both success paths and failure scenarios for comprehensive coverage

In scope:
- All agent component implementations (AmpAgent, OpenCodeAgent, JulesAgent, JunieAgent, WindsurfAgent, WarpAgent, and future agents)
- CLI handler functions and command processing logic
- Core business logic that requires isolation testing
- Error handling and validation logic for agent operations

Out of scope:
- Integration tests that require multiple components or external dependencies
- End-to-end tests that validate full system behavior
- Performance or load testing scenarios
- Manual testing procedures or exploratory testing

## Rationale

- The pattern appears in 8 files with 92.19% significance, indicating strong adoption and proven value across the codebase
- TypeScript unit tests provide type safety during test development, catching errors earlier and improving test reliability
- Consistent file naming and directory structure (tests/unit/agents/, tests/unit/cli/) enables developers to quickly locate and understand test coverage
- Separating unit tests from integration tests allows for faster test execution and clearer separation of concerns
- The pattern supports multiple agent implementations, demonstrating scalability and maintainability as new agents are added

## Consequences

Positive:
- Developers can quickly identify test coverage for any agent by following predictable file naming conventions
- Type safety in tests reduces runtime errors and improves refactoring confidence
- Isolated unit tests enable rapid feedback during development without requiring full system setup
- Consistent testing patterns reduce cognitive load when switching between different agent implementations
- New team members can understand testing expectations by examining existing test files

Negative:
- Requires discipline to maintain consistent naming and structure as the codebase grows
- TypeScript test files may have slightly longer build times compared to plain JavaScript
- Developers must understand the distinction between unit and integration tests to place tests correctly
- Additional tooling configuration may be required to support TypeScript in the test environment

## Alternatives

- Use JavaScript (.test.js) instead of TypeScript for test files (rejected)
  Rejected because: JavaScript tests would lose type safety benefits and create inconsistency with the TypeScript codebase, increasing the likelihood of test errors and reducing refactoring confidence
  When valid: Only valid for legacy codebases without TypeScript or when type safety is explicitly not required
- Co-locate tests with source files (e.g., agents/AmpAgent.test.ts next to agents/AmpAgent.ts) (rejected)
  Rejected because: Separate test directory structure provides clearer separation of concerns and makes it easier to exclude tests from production builds
  When valid: Valid for smaller projects or when build tooling makes test exclusion trivial
- Combine unit and integration tests in a single tests/ directory without subdirectories (rejected)
  Rejected because: Mixing test types makes it difficult to run fast unit tests independently and obscures the testing strategy
  When valid: Valid only for very small projects with minimal test suites

## Risks

- Test files may become outdated if not maintained alongside source code changes
  Mitigation: Implement CI checks that fail if agent files exist without corresponding test files, and enforce code review standards requiring test updates
  Owner: Engineering team
- Inconsistent test quality across different agent implementations despite consistent structure
  Mitigation: Establish test coverage thresholds and code review guidelines that verify test completeness, not just test existence
  Owner: Engineering team
- TypeScript configuration drift between test and source environments may cause test failures
  Mitigation: Use shared TypeScript configuration (tsconfig.json) and regularly validate test environment setup in CI
  Owner: DevOps team

## Implementation Notes

- When creating a new agent, immediately create the corresponding test file in tests/unit/agents/{AgentName}.test.ts before implementing the agent logic (TDD approach)
- Use a test template or generator to ensure consistent test structure across all agent implementations
- Configure your test runner (Jest, Vitest, etc.) to recognize .test.ts files and compile TypeScript appropriately
- Organize test cases within each file using describe blocks for different aspects of agent behavior (initialization, execution, error handling)
- Ensure your IDE or editor is configured to run TypeScript tests directly for rapid feedback during development

## Continuation Context


Verify commands:
- find tests/unit/agents -name '*.test.ts' | wc -l
- grep -r 'describe\|it\|test' tests/unit/agents/*.test.ts | head -5
- ls tests/unit/cli/*.test.ts 2>/dev/null || echo 'No CLI tests found'

Accept when:
- All agent implementation files in the agents/ directory have corresponding .test.ts files in tests/unit/agents/
- Test files use TypeScript and follow the naming convention {ComponentName}.test.ts
- Tests are organized in the tests/unit/ directory hierarchy with appropriate subdirectories (agents/, cli/)

## Enforcement

- Verified by: Automated CI pipeline checks that verify test file existence for each agent implementation
- Verified by: Code review process that requires test files for new agent implementations
- Verified by: Static analysis tools that validate test file naming conventions and directory structure
- Violation handling: Pull requests without corresponding test files for new agents are automatically flagged and blocked from merging
- Violation handling: CI builds fail if test coverage drops below established thresholds
- Violation handling: Code review checklist includes verification of test file structure and naming compliance
- Exception process: Exceptions require explicit justification in the pull request description explaining why tests cannot be provided
- Exception process: Technical lead or architect must approve any exception to the testing pattern
- Exception process: Exceptions are tracked and reviewed quarterly to identify systemic issues or needed pattern updates