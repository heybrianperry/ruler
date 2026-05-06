# Standardize Unit Testing for Agent Implementations: Unit Tests Organized

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent implementations and CLI handler modules within the codebase.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, JunieAgent, AmpAgent, JulesAgent, WarpAgent, WindsurfAgent) that require consistent testing approaches to ensure reliability and maintainability.
- Unit tests have been established in a dedicated tests/unit directory structure, separating test code from implementation code and organizing tests by component type (agents, cli).
- Pattern detection identified 8 files with 92.19% confidence showing consistent unit testing practices across agent implementations and CLI handlers.
- The testing pattern emerged organically across multiple agent implementations, indicating a de facto standard that should be formalized to maintain consistency as new agents are added.
- TypeScript-based testing infrastructure is in place, requiring test files to follow .test.ts naming conventions for proper test discovery and execution.

## Problem Statement

Without a formalized unit testing standard for agent implementations and CLI handlers, the codebase risks inconsistent test coverage, varying test quality, and difficulty maintaining test suites as new agents are added. The lack of explicit testing requirements could lead to untested agent code being merged, reducing system reliability and making refactoring more dangerous.

## Decision

1. MUST: Unit tests MUST be organized in a tests/unit/ directory structure that mirrors the source code organization

## Policy Block

- MUST Unit tests MUST be organized in a tests/unit/ directory structure that mirrors the source code organization

In scope:
- All agent implementation classes (e.g., OpenCodeAgent, JunieAgent, AmpAgent, JulesAgent, WarpAgent, WindsurfAgent)
- CLI handler modules and command processing logic
- Core business logic and domain models that require unit-level verification
- Error handling and validation logic for agents and handlers

Out of scope:
- Integration tests that verify multi-component interactions
- End-to-end tests that exercise the full application stack
- Performance and load testing scenarios
- Manual testing and exploratory testing activities

Exceptions:
- EXC-001: Prototype or experimental agent implementations that are explicitly marked as non-production and isolated from the main codebase

## Rationale

- Pattern detection identified 8 files with 92.19% significance showing consistent adoption of this testing structure across multiple agent implementations, indicating strong team consensus and proven effectiveness.
- Organizing unit tests in a dedicated tests/unit/ directory with mirrored structure improves discoverability, makes test maintenance easier, and clearly separates test code from production code.
- Standardizing the .test.ts naming convention ensures compatibility with common TypeScript testing frameworks (Jest, Vitest, etc.) and enables automated test discovery in CI/CD pipelines.
- Requiring unit tests for all agents ensures consistent quality standards and reduces the risk of regressions when modifying agent behavior or adding new agent types.

## Consequences

Positive:
- Consistent test coverage across all agent implementations improves overall system reliability and makes it easier to identify untested code
- Standardized test organization reduces cognitive load for developers working across different agents and modules
- Automated test discovery through naming conventions enables seamless CI/CD integration and prevents tests from being accidentally skipped
- Clear testing standards make onboarding new developers easier and establish quality expectations for contributions

Negative:
- Requires upfront investment in writing unit tests for each new agent, potentially slowing initial development velocity
- Maintaining parallel test file structure adds overhead when reorganizing source code directories
- Strict naming conventions may feel restrictive for developers accustomed to different testing patterns
- May lead to over-emphasis on unit test coverage metrics at the expense of integration and end-to-end testing

## Alternatives

- Co-locate test files with source code (e.g., Agent.ts and Agent.test.ts in same directory) (rejected)
  Rejected because: Mixing test and source files in the same directory clutters the source tree and makes it harder to exclude tests from production builds. The detected pattern shows strong preference for separated test directories.
  When valid: May be appropriate for very small utility modules or when using a framework that strongly encourages co-location
- Use __tests__ directories adjacent to source files following Jest conventions (rejected)
  Rejected because: The detected pattern shows consistent use of a centralized tests/unit/ structure rather than distributed __tests__ directories. Changing this would break existing conventions.
  When valid: Could be considered for new projects or during a major testing infrastructure refactor if team consensus supports the change
- Make unit testing optional with coverage thresholds enforced at the module level (rejected)
  Rejected because: Optional testing leads to inconsistent coverage and makes it difficult to maintain quality standards. The high confidence (92.19%) of the detected pattern indicates testing is already a de facto requirement.
  When valid: Not recommended for production code; may be acceptable for internal tools or scripts with limited scope

## Risks

- Developers may write superficial tests just to satisfy the requirement without achieving meaningful coverage
  Mitigation: Implement code review guidelines that check for test quality, not just presence. Use coverage tools to identify untested code paths. Provide examples of good agent tests.
  Owner: Engineering team leads and code reviewers
- Test maintenance burden may grow significantly as the number of agents increases, leading to brittle or outdated tests
  Mitigation: Establish shared test utilities and fixtures for common agent testing patterns. Regularly refactor tests to reduce duplication. Include test maintenance in sprint planning.
  Owner: Engineering team
- Strict file naming and location requirements may cause confusion or friction during rapid prototyping phases
  Mitigation: Document clear examples and provide test file templates. Allow temporary exceptions for prototype code with explicit tech lead approval and follow-up tasks.
  Owner: Tech leads and documentation team

## Implementation Notes

- Create a test file template for new agents that includes common test structure, imports, and placeholder test cases to accelerate test creation
- Configure the testing framework to automatically discover and run all .test.ts files in the tests/unit/ directory tree
- Add pre-commit hooks or CI checks to verify that new agent files have corresponding test files before allowing merge
- Document testing patterns and best practices in a TESTING.md guide, including examples from existing agent tests like OpenCodeAgent.test.ts and JunieAgent.test.ts

## Continuation Context


Verify commands:
- find tests/unit/agents -name '*.test.ts' | wc -l
- grep -r 'describe\|test\|it' tests/unit/agents/ | wc -l
- npm test -- --coverage --coverageDirectory=coverage

Accept when:
- All agent implementation files in the source tree have corresponding .test.ts files in tests/unit/agents/
- Test discovery finds and executes all unit tests without manual configuration
- Code coverage reports show unit test execution for agent modules and CLI handlers

## Enforcement

- Verified by: Automated CI pipeline checks that verify presence of test files for each agent implementation
- Verified by: Code review checklist requiring reviewers to confirm unit tests are present and meaningful
- Verified by: Coverage reports generated on each pull request showing unit test execution
- Violation handling: Pull requests without corresponding unit tests for new agents are automatically flagged and blocked from merging
- Violation handling: CI build fails if test file naming conventions are not followed or tests are not discoverable
- Violation handling: Monthly audit reports identify agents with missing or inadequate unit test coverage for remediation
- Exception process: Developer submits exception request to tech lead with justification for why unit tests cannot be provided
- Exception process: Tech lead reviews request and either approves with documented follow-up task or requests test implementation
- Exception process: Approved exceptions are tracked in a technical debt register with target resolution dates