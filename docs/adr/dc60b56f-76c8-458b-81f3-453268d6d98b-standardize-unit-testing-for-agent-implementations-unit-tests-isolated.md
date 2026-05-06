# Standardize Unit Testing for Agent Implementations: Unit Tests Isolated

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent implementations and test suites within the codebase.

## Context

- The codebase contains multiple agent implementations (AmpAgent, OpenCodeAgent, JulesAgent, JunieAgent, WindsurfAgent, WarpAgent) that require consistent testing approaches
- Pattern detection identified 8 test files with 92.19% confidence showing a standardized unit testing structure for agent components
- Developer tooling patterns indicate a mature testing infrastructure with dedicated test suites for each agent type and CLI handlers
- The presence of invalid-agent.test.ts suggests deliberate testing of error conditions and edge cases
- Consistent test file naming and organization patterns emerged across all agent implementations, indicating an established testing convention

## Problem Statement

Without standardized unit testing practices for agent implementations, the codebase risks inconsistent test coverage, unpredictable agent behavior, and difficulty maintaining quality across multiple agent types. A unified testing approach ensures all agents meet the same quality standards and behavioral expectations.

## Decision

1. SHOULD: Unit tests SHOULD be isolated from integration concerns and focus on individual agent behavior

## Policy Block

- SHOULD Unit tests SHOULD be isolated from integration concerns and focus on individual agent behavior

In scope:
- All agent implementations (AmpAgent, OpenCodeAgent, JulesAgent, JunieAgent, WindsurfAgent, WarpAgent, and future agents)
- CLI handler implementations
- Core library functionality requiring unit test coverage
- Error handling and invalid input scenarios

Out of scope:
- Integration tests that span multiple components
- End-to-end tests requiring full system setup
- Performance and load testing
- Manual testing procedures

## Rationale

- Pattern detection across 8 files with 92.19% confidence indicates this is an established and successful practice in the codebase
- Consistent test file organization improves developer productivity by making test locations predictable
- Dedicated unit tests for each agent ensure behavioral consistency and catch regressions early in development
- The presence of invalid-agent.test.ts demonstrates commitment to comprehensive error handling validation

## Consequences

Positive:
- Consistent test coverage across all agent implementations ensures uniform quality standards
- Predictable test file locations reduce cognitive load for developers navigating the codebase
- Early detection of regressions and behavioral changes through comprehensive unit testing
- Clear separation between unit and integration tests improves test execution speed and reliability

Negative:
- Requires discipline to maintain test file naming conventions and structure as new agents are added
- Initial setup cost for creating comprehensive test suites for each new agent implementation
- May lead to test duplication if common agent behaviors are not abstracted into shared test utilities
- Strict adherence to unit testing patterns may delay rapid prototyping of experimental agents

## Alternatives

- Co-locate tests with implementation files instead of separate tests/unit directory (rejected)
  Rejected because: Separating tests from implementation provides cleaner project structure and allows for different build/deployment configurations
  When valid: May be reconsidered for very small utility modules where co-location improves discoverability
- Use integration tests exclusively without dedicated unit tests (rejected)
  Rejected because: Integration tests are slower and less precise for catching agent-specific behavioral issues; unit tests provide faster feedback
  When valid: Integration tests remain valuable as a complement to unit tests, not a replacement
- Implement shared test base classes for common agent testing patterns (deferred)
  Rejected because: Not rejected, but deferred pending analysis of common patterns across existing test suites
  When valid: Should be reconsidered once sufficient test duplication is observed across agent implementations

## Risks

- Test coverage may become inconsistent as new developers add agents without following established patterns
  Mitigation: Implement automated checks in CI to verify test file existence and naming conventions; provide clear documentation and templates
  Owner: Engineering team
- Unit tests may become brittle if they test implementation details rather than behavior
  Mitigation: Establish code review guidelines emphasizing behavioral testing; provide examples of good vs brittle tests
  Owner: Engineering team
- Test maintenance burden may grow as agent count increases
  Mitigation: Invest in shared test utilities and fixtures; regularly refactor common patterns into reusable components
  Owner: Engineering team

## Implementation Notes

- Use the existing test files (AmpAgent.test.ts, OpenCodeAgent.test.ts, etc.) as templates when creating tests for new agents
- Ensure test file names exactly match the agent implementation name with .test.ts suffix
- Include both positive test cases (valid agent operations) and negative test cases (error handling, invalid inputs)
- Leverage TypeScript's type system in tests to catch interface changes early
- Consider creating a test utilities module for common agent testing patterns to reduce duplication

## Continuation Context


Verify commands:
- find tests/unit/agents -name '*.test.ts' | wc -l
- grep -r 'describe\|it\|test' tests/unit/agents/ | wc -l
- npm test -- --coverage --testPathPattern=tests/unit

Accept when:
- Each agent implementation in the codebase has a corresponding test file in tests/unit/agents/
- Test coverage reports show >80% coverage for agent implementations
- All test files follow the naming convention {ComponentName}.test.ts and execute successfully in CI

## Enforcement

- Verified by: Automated CI pipeline checks for test file existence matching agent implementations
- Verified by: Code coverage reports generated on each pull request
- Verified by: Code review process verifying test quality and adherence to patterns
- Violation handling: Pull requests without corresponding test files for new agents are blocked from merging
- Violation handling: Coverage drops below threshold trigger CI failures and require remediation
- Violation handling: Code review feedback requests test additions or improvements before approval
- Exception process: Exceptions for experimental or prototype agents must be documented in PR description with timeline for adding tests
- Exception process: Technical debt tickets must be created for any deferred test implementation
- Exception process: Exceptions require approval from team lead or architect