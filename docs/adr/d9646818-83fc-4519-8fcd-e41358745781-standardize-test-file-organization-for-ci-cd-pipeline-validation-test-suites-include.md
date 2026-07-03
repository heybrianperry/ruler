# Standardize Test File Organization for CI/CD Pipeline Validation: Test Suites Include

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all test files within the CI/CD pipeline and applies to all agent-related test suites, configuration validation tests, and integration tests.

## Context

- The codebase contains 16 test files with consistent patterns for validating agent configurations, MCP (Model Context Protocol) settings, and integration behaviors
- Tests are organized to validate security-critical input validation scenarios including HTTP type validation, configuration overrides, and nested behavior propagation
- The pattern emerged from the need to ensure consistent CI/CD validation across multiple agent types (ZedAgent, CodexCliAgent, FirebenderAgent, CrushAgent) and configuration scenarios
- Test files demonstrate a systematic approach to validating both unit-level agent behavior and integration-level configuration propagation
- The pattern shows strong consistency (92.50% confidence) across 16 files, indicating an established architectural practice rather than ad-hoc testing

## Problem Statement

Without standardized test organization and validation patterns in CI/CD pipelines, configuration errors, security vulnerabilities in input validation, and agent behavior inconsistencies may not be detected until runtime, leading to production failures and security incidents.

## Decision

1. SHOULD: Test suites SHOULD include specific test cases for agent-specific disable functionality and configuration overwrite behavior

## Policy Block

- SHOULD Test suites SHOULD include specific test cases for agent-specific disable functionality and configuration overwrite behavior

In scope:
- All test files within the tests/ directory structure
- Unit tests for agent implementations
- Integration tests for configuration propagation
- Configuration validation tests for MCP, HTTP, and type field validation
- CI/CD pipeline test execution

Out of scope:
- End-to-end tests that span multiple services
- Performance or load testing scenarios
- Manual testing procedures
- Production monitoring and alerting
- Documentation-only changes without code impact

Exceptions:
- EX-001: Legacy agent implementations that are scheduled for deprecation within the current quarter
- EX-002: Experimental or prototype agents in feature branches not yet merged to main

## Rationale

- The pattern was detected across 16 files with 92.50% confidence, indicating a well-established and consistent practice that has proven effective
- Standardized test organization enables early detection of configuration errors and security vulnerabilities in the CI/CD pipeline before code reaches production
- The focus on input validation testing (security.input_validation facet) addresses critical security concerns around configuration injection and malformed inputs
- Consistent test structure across multiple agent types (Zed, Codex, Firebender, Crush) demonstrates scalability and maintainability of the pattern

## Consequences

Positive:
- Early detection of configuration errors and security vulnerabilities during CI/CD execution
- Consistent test structure reduces cognitive load for developers working across different agent implementations
- Comprehensive validation coverage for MCP configurations, HTTP types, and nested behaviors increases system reliability
- Clear separation between unit and integration tests improves test execution speed and debugging efficiency

Negative:
- Requires initial investment to establish test infrastructure and patterns for new agent types
- May increase test suite execution time as comprehensive validation is added
- Developers must learn and follow specific naming conventions and organizational patterns
- Maintenance overhead increases with the number of agent-specific test files

## Alternatives

- Consolidate all tests into a single monolithic test file per agent without separation of unit/integration concerns (rejected)
  Rejected because: Reduces test execution efficiency, makes debugging harder, and violates separation of concerns principles. The current pattern's clear separation enables faster feedback loops.
  When valid: Only appropriate for very simple agents with minimal configuration surface area
- Rely on end-to-end tests only without dedicated unit and integration test layers (rejected)
  Rejected because: E2E tests are slower, more brittle, and provide delayed feedback. The layered testing approach catches issues earlier in the development cycle.
  When valid: May be acceptable for proof-of-concept implementations not intended for production
- Use property-based testing frameworks for configuration validation instead of explicit test cases (deferred)
  Rejected because: Not rejected but deferred for future consideration. Could complement existing tests but requires evaluation of tooling and team expertise.
  When valid: Could be valuable for discovering edge cases in complex configuration schemas

## Risks

- Test suite becomes too large and slow, impacting developer productivity and CI/CD pipeline performance
  Mitigation: Implement test parallelization, use test impact analysis to run only affected tests, and establish test execution time budgets
  Owner: Engineering team with CI/CD platform support
- Inconsistent application of testing patterns as new developers join or new agent types are added
  Mitigation: Create test templates and scaffolding tools, include testing patterns in onboarding documentation, and enforce via code review checklists
  Owner: Engineering team leads
- Tests may become outdated as configuration schemas evolve, leading to false confidence
  Mitigation: Implement schema validation tests that fail when schemas change without corresponding test updates, and include test maintenance in definition of done
  Owner: Engineering team

## Implementation Notes

- Use test file templates that include standard sections for input validation, configuration override testing, and nested behavior validation
- Establish naming conventions: tests/unit/agents/{AgentName}.test.ts for unit tests, tests/integration/{feature-name}.test.ts for integration tests, and tests/{config-aspect}.test.ts for configuration validation
- Implement test helpers for common validation patterns (MCP config validation, HTTP type checking, agent-specific disable logic) to reduce duplication
- Configure CI/CD pipeline to run unit tests first for fast feedback, followed by integration tests, with clear failure reporting that maps to the test organization structure

## Continuation Context


Verify commands:
- find tests/ -name '*.test.ts' | grep -E '(unit/agents/|integration/|claude-|gemini-|mcp-)' | wc -l
- grep -r 'describe.*Agent' tests/unit/agents/ | wc -l
- grep -r 'input.*validation\|type.*field\|MCP.*config' tests/ | wc -l

Accept when:
- All agent implementations have corresponding test files in tests/unit/agents/ directory
- Configuration validation tests exist for all security-critical input fields (type, HTTP config, MCP settings)
- Test organization follows the three-tier structure: unit tests, integration tests, and configuration tests with clear naming conventions

## Enforcement

- Verified by: CI/CD pipeline test execution with mandatory pass requirements
- Verified by: Code review checklist verification for new agent implementations
- Verified by: Automated test coverage reporting with minimum thresholds
- Verified by: Pre-commit hooks that validate test file naming conventions
- Violation handling: CI/CD pipeline blocks merge if required tests are missing or failing
- Violation handling: Code review process flags missing test coverage for new agents or configuration changes
- Violation handling: Automated alerts sent to team leads when test coverage drops below thresholds
- Violation handling: Pull requests without corresponding tests are marked as incomplete and returned to author
- Exception process: Developer submits exception request via pull request comment with justification
- Exception process: Engineering lead reviews and approves/denies based on risk assessment
- Exception process: Approved exceptions are documented in code comments and tracked in technical debt backlog
- Exception process: All exceptions require follow-up tickets for remediation within defined timeline