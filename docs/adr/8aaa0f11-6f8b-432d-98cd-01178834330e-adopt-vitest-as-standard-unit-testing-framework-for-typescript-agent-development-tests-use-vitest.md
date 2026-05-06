# Adopt Vitest as Standard Unit Testing Framework for TypeScript Agent Development: Tests Use Vitest

Status: proposed
Date: 2025-01-10
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all TypeScript test file development in the tests/unit directory and applies to all agent test implementations.

## Context

- The codebase contains multiple agent implementations (OpenCodeAgent, JunieAgent, AmpAgent, JulesAgent, WarpAgent, WindsurfAgent) that require consistent unit testing approaches
- Pattern signature 4ae03170f2fea77bb3b86d4a37689c12 detected across 8 test files with 92.19% confidence, indicating a standardized testing framework adoption
- TypeScript-based agent development requires a modern, fast testing framework with native TypeScript support and minimal configuration overhead
- The test.frameworks facet indicates a deliberate architectural choice for developer tooling standardization across the project
- Unit tests are located in a structured tests/unit directory hierarchy, suggesting organized test architecture requiring consistent tooling

## Problem Statement

Without a standardized testing framework, agent development teams face inconsistent test execution environments, varying assertion APIs, incompatible mocking strategies, and fragmented developer experience. This leads to increased onboarding time, reduced test reliability, and difficulty maintaining test suites across multiple agent implementations.

## Decision

1. MAY: Tests MAY use Vitest's snapshot testing features for complex output validation when appropriate

## Policy Block

- MAY Tests MAY use Vitest's snapshot testing features for complex output validation when appropriate

In scope:
- All TypeScript unit tests in the tests/unit directory
- Agent implementation tests (OpenCodeAgent, JunieAgent, AmpAgent, JulesAgent, WarpAgent, WindsurfAgent)
- CLI handler tests and utility function tests
- New test files created for TypeScript components

Out of scope:
- Integration tests that may require different testing frameworks
- End-to-end tests using browser automation or API testing tools
- Performance benchmarking tests that may use specialized tooling
- Legacy JavaScript tests not yet migrated to TypeScript

Exceptions:
- EXC-001: Testing framework-specific features that require alternative testing tools (e.g., React Testing Library for component tests)

## Rationale

- Pattern detected across 8 files with 92.19% confidence indicates strong existing adoption and proven effectiveness in the codebase
- Vitest provides native TypeScript support, fast execution with ESM compatibility, and a Jest-compatible API reducing migration friction
- Standardizing on a single testing framework reduces cognitive load, simplifies CI/CD configuration, and improves test maintainability across agent implementations
- The test.frameworks facet classification confirms this is a deliberate architectural decision for developer tooling standardization

## Consequences

Positive:
- Consistent testing API and patterns across all agent implementations reduces onboarding time for new developers
- Fast test execution with Vitest's native ESM support and parallel test running improves developer productivity
- Unified mocking and assertion strategies simplify test maintenance and reduce framework-specific knowledge requirements
- Strong TypeScript integration provides better type safety and IDE support during test development

Negative:
- Teams familiar with alternative frameworks (Jest, Mocha) must learn Vitest-specific APIs and configuration
- Existing tests using different frameworks require migration effort and potential rewriting
- Framework lock-in creates dependency on Vitest's continued maintenance and community support
- Some advanced testing scenarios may require workarounds if Vitest lacks specific features available in other frameworks

## Alternatives

- Continue using Jest as the primary testing framework (rejected)
  Rejected because: Jest has slower TypeScript execution requiring ts-jest transformation, lacks native ESM support, and the pattern detection shows clear migration to Vitest across the codebase
  When valid: For projects with extensive Jest test suites where migration cost outweighs Vitest benefits
- Allow multiple testing frameworks based on team preference (rejected)
  Rejected because: Multiple frameworks increase maintenance burden, fragment developer knowledge, complicate CI/CD pipelines, and contradict the detected standardization pattern
  When valid: In polyglot codebases with distinct technology stacks requiring specialized testing tools
- Use native Node.js test runner for unit tests (rejected)
  Rejected because: Node.js native test runner lacks maturity, has limited ecosystem support, and provides fewer features compared to Vitest's comprehensive testing capabilities
  When valid: For minimal dependency projects prioritizing zero external test dependencies

## Risks

- Vitest framework deprecation or lack of maintenance could require future migration to alternative testing frameworks
  Mitigation: Monitor Vitest community health, maintain test abstraction layers where possible, and document migration paths in case of framework changes
  Owner: Engineering Team
- Incomplete migration from legacy testing frameworks may result in inconsistent test execution and maintenance overhead
  Mitigation: Create migration guide, prioritize test file migration in sprint planning, and establish deadline for complete framework consolidation
  Owner: Engineering Team
- Vitest-specific features may create testing patterns that are difficult to migrate if framework change becomes necessary
  Mitigation: Prefer standard testing patterns over framework-specific features, document framework-specific usage, and maintain test portability guidelines
  Owner: Engineering Team

## Implementation Notes

- Install Vitest as a dev dependency and configure vitest.config.ts with TypeScript support and test file patterns
- Create test file templates for common agent testing scenarios to ensure consistent test structure across implementations
- Update CI/CD pipelines to execute tests using 'vitest run' command with appropriate coverage thresholds
- Document Vitest-specific patterns in testing guidelines including mocking strategies, async test handling, and snapshot usage
- Establish test file naming convention (*.test.ts) and directory structure (tests/unit/{component}) in project documentation

## Continuation Context


Verify commands:
- grep -r "from 'vitest'" tests/unit/ | wc -l
- find tests/unit -name '*.test.ts' -type f | wc -l
- grep -r "describe\|it\|test\|expect" tests/unit/agents/*.test.ts | head -5
- cat package.json | grep -A 2 '"vitest"'

Accept when:
- All test files in tests/unit directory import testing utilities from 'vitest' package
- Test files follow naming convention *.test.ts and are organized in tests/unit/{component} structure
- Package.json includes vitest as a dev dependency and test scripts execute vitest commands
- Agent test files (OpenCodeAgent, JunieAgent, AmpAgent, etc.) successfully execute using vitest runner

## Enforcement

- Verified by: CI pipeline test execution stage verifies all tests run using Vitest framework
- Verified by: Code review checklist includes verification of test framework compliance for new test files
- Verified by: Automated linting rules detect non-Vitest test imports in tests/unit directory
- Violation handling: CI pipeline fails if test files use non-Vitest testing frameworks in tests/unit directory
- Violation handling: Code review blocks merge requests containing test files that violate framework standards
- Violation handling: Automated alerts notify team leads of framework violations detected in pull requests
- Exception process: Developer submits exception request to Tech Lead with documented justification for alternative framework
- Exception process: Tech Lead reviews technical rationale and approves/rejects based on project standards and specific use case
- Exception process: Approved exceptions are documented in test file header comments with approval reference and expiration review date