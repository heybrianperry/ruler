# Standardize Unit Testing Structure for Agent Components: Teams Organize Agent

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent component development and testing activities.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, WarpAgent, WindsurfAgent, JunieAgent, JulesAgent) that require consistent testing approaches
- Unit tests for agent components are organized in a dedicated tests/unit/agents/ directory structure, indicating a deliberate architectural choice for test organization
- All five agent test files follow a consistent naming pattern (*.test.ts) and location pattern, suggesting a standardized testing convention across the team
- The pattern has 92.50% confidence with 5 supporting files, indicating strong consistency in the testing approach
- Agent components represent critical business logic that requires isolated unit testing to ensure reliability and maintainability

## Problem Statement

Without a standardized unit testing structure for agent components, teams may implement inconsistent testing approaches, leading to gaps in test coverage, difficulty in maintaining tests, and reduced confidence in agent behavior. A consistent testing pattern ensures all agent implementations are validated uniformly and maintainably.

## Decision

1. MAY: Teams MAY organize agent tests into subdirectories within tests/unit/agents/ if the number of agents grows significantly

## Policy Block

- MAY Teams MAY organize agent tests into subdirectories within tests/unit/agents/ if the number of agents grows significantly

In scope:
- All agent component classes and their unit tests
- New agent implementations added to the system
- Refactored or modified existing agent components
- Test infrastructure and tooling for agent testing

Out of scope:
- Integration tests for agents (should be in separate integration test directory)
- End-to-end tests involving multiple agents
- Performance or load tests for agents
- Non-agent component unit tests

## Rationale

- Consistent test organization enables developers to quickly locate and understand tests for any agent component, reducing cognitive load and onboarding time
- The pattern signature (0e77cc830f33495c7a206c943747671a) detected across 5 agent test files demonstrates this is an established practice worth codifying
- Standardized naming conventions prevent confusion and ensure test discovery tools can automatically identify and run agent unit tests
- Isolating unit tests in a dedicated directory structure supports separation of concerns and enables targeted test execution during development

## Consequences

Positive:
- Developers can immediately locate unit tests for any agent by following the predictable naming and directory structure
- Automated test runners and CI/CD pipelines can reliably discover and execute agent unit tests
- New team members can quickly understand the testing approach by examining any existing agent test as a reference
- Consistent structure facilitates code reviews and ensures no agent is deployed without corresponding unit tests

Negative:
- Rigid directory structure may require refactoring if the number of agents grows to hundreds
- Teams must maintain discipline to follow the convention even under time pressure
- Migration of existing non-conformant tests requires effort and coordination
- Additional tooling configuration may be needed to ensure test paths are correctly resolved

## Alternatives

- Co-locate tests with agent source files in the same directory (rejected)
  Rejected because: Mixing test and source files reduces clarity and makes it harder to exclude tests from production builds. The detected pattern shows deliberate separation.
  When valid: For very small projects with fewer than 5 components where simplicity outweighs organization
- Use a flat tests/ directory without subdirectories for unit/integration separation (rejected)
  Rejected because: Flat structure becomes unmanageable as test count grows and prevents targeted execution of unit vs integration tests
  When valid: Never recommended for projects with multiple test types
- Organize tests by feature rather than by component type (deferred)
  When valid: Could be considered if agents become part of larger feature modules, but current pattern shows component-based organization is working well

## Risks

- Developers may forget to create unit tests for new agents, especially under deadline pressure
  Mitigation: Implement CI checks that fail if an agent class exists without a corresponding test file. Add PR templates that include test file checklist.
  Owner: Engineering team and CI/CD maintainers
- Test file naming inconsistencies may emerge over time as different developers contribute
  Mitigation: Add linting rules or pre-commit hooks that validate test file naming conventions. Document the pattern in contribution guidelines.
  Owner: Engineering team
- As the number of agents grows, the flat tests/unit/agents/ directory may become difficult to navigate
  Mitigation: Monitor directory size and establish threshold (e.g., 20+ agents) to trigger discussion of subdirectory organization
  Owner: Technical lead

## Implementation Notes

- Create a test template or generator script that scaffolds new agent test files with the correct naming and location
- Update project documentation and contribution guidelines to explicitly describe the agent testing structure
- Configure test runners (Jest, Vitest, etc.) to recognize the tests/unit/agents/ path pattern
- Consider adding a script that validates all agent classes have corresponding test files and reports gaps

## Continuation Context


Verify commands:
- find tests/unit/agents -name '*.test.ts' | wc -l
- ls tests/unit/agents/*.test.ts 2>/dev/null || echo 'No agent tests found'
- test -d tests/unit/agents && echo 'Agent test directory exists' || echo 'Missing agent test directory'

Accept when:
- The tests/unit/agents/ directory exists and contains .test.ts files
- Each agent implementation class has a corresponding {AgentName}.test.ts file in tests/unit/agents/
- Test files can be discovered and executed by the project's test runner without additional configuration

## Enforcement

- Verified by: Automated CI pipeline checks that validate test file existence for each agent
- Verified by: Code review process that verifies new agents include corresponding unit tests
- Verified by: Pre-commit hooks that validate test file naming conventions
- Violation handling: CI build fails if an agent class is detected without a corresponding unit test file
- Violation handling: Pull requests are blocked from merging if test structure violations are detected
- Violation handling: Automated notifications are sent to developers when violations are detected
- Exception process: Exceptions must be documented in a TESTING_EXCEPTIONS.md file with justification
- Exception process: Technical lead approval required for any agent without unit tests
- Exception process: Exception requests must include a remediation plan with timeline