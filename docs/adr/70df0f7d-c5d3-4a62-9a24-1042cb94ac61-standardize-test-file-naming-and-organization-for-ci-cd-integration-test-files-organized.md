# Standardize Test File Naming and Organization for CI/CD Integration: Test Files Organized

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all test file creation and CI/CD pipeline configuration activities.

## Context

- The codebase contains 38 test files following a consistent naming pattern with .test.ts extensions, indicating a mature testing infrastructure integrated with CI/CD pipelines
- Test files are organized across multiple test types (unit, integration, e2e) with clear directory structure separating concerns and test scopes
- The pattern shows strong adoption across agent testing, MCP (Model Context Protocol) testing, and integration testing scenarios, suggesting this is a project-wide standard
- Consistent test file naming enables automated test discovery, parallel execution, and selective test running in CI/CD workflows
- The high confidence (92.48%) and support count (38 files) indicates this is an established architectural pattern rather than an emerging practice

## Problem Statement

Without standardized test file naming conventions and organizational structure, CI/CD pipelines cannot reliably discover, categorize, and execute tests. This leads to inconsistent test coverage reporting, difficulty in running specific test suites, and challenges in maintaining test infrastructure as the codebase scales.

## Decision

1. MUST: Test files MUST be organized into dedicated test directories (tests/unit, tests/integration, tests/e2e) based on their test scope

## Policy Block

- MUST Test files MUST be organized into dedicated test directories (tests/unit, tests/integration, tests/e2e) based on their test scope

In scope:
- All TypeScript test files in the project
- Unit tests, integration tests, and end-to-end tests
- Agent testing files, MCP testing files, and feature testing files
- CI/CD pipeline test discovery and execution configuration

Out of scope:
- Non-TypeScript test files (if any exist in other languages)
- Test utilities and helper files that are not executable tests
- Mock data files and test fixtures
- Build scripts and CI/CD configuration files themselves

Exceptions:
- EXC-001: Legacy test files exist from before this standard was adopted
- EXC-002: Third-party or vendor-provided test files use different conventions

## Rationale

- The pattern detection identified 38 files with 92.48% confidence, demonstrating this is an established and successful practice in the codebase
- Consistent test file naming enables CI/CD tools to automatically discover and execute tests without manual configuration updates
- Organizing tests by scope (unit/integration/e2e) allows selective test execution, reducing CI/CD pipeline duration and enabling faster feedback loops
- The .test.ts extension is a widely recognized convention in the TypeScript/JavaScript ecosystem, making the codebase more accessible to new developers

## Consequences

Positive:
- Automated test discovery eliminates manual pipeline configuration when new tests are added
- Clear test organization enables parallel test execution and selective test running based on change scope
- Consistent naming improves developer experience and reduces cognitive load when navigating the test suite
- Better test coverage reporting and metrics collection through standardized file patterns

Negative:
- Requires migration effort for any existing tests that don't follow the convention
- May require updates to existing CI/CD pipeline configurations to adopt the standardized glob patterns
- Developers must learn and remember the organizational structure and naming conventions
- Potential confusion if similar components exist in different test scopes (e.g., unit vs integration)

## Alternatives

- Use .spec.ts extension instead of .test.ts for test files (rejected)
  Rejected because: The existing codebase has already standardized on .test.ts with 38 files, and changing would require significant migration effort without clear benefit
  When valid: Could be considered for new projects starting from scratch
- Co-locate tests with source files instead of separate test directories (rejected)
  Rejected because: Separate test directories provide clearer separation of concerns and make it easier to exclude tests from production builds
  When valid: May be appropriate for small utility libraries where co-location improves discoverability
- Use a single flat tests/ directory without subdirectories for test types (rejected)
  Rejected because: Organizing by test scope (unit/integration/e2e) enables selective test execution and clearer test categorization as the suite grows
  When valid: Acceptable only for very small projects with fewer than 10 test files

## Risks

- Inconsistent application of naming conventions by developers unfamiliar with the standard
  Mitigation: Implement automated linting rules and pre-commit hooks to enforce test file naming patterns
  Owner: Engineering team
- CI/CD pipeline failures if glob patterns don't match all test file variations
  Mitigation: Regularly audit test discovery patterns and maintain comprehensive documentation of naming conventions
  Owner: DevOps team
- Test organization becomes unclear as the codebase scales and new test types emerge
  Mitigation: Establish clear guidelines for when to create new test directories and document decision criteria
  Owner: Architecture team

## Implementation Notes

- Configure test runners (Jest, Vitest, etc.) to use the glob pattern **/*.test.ts for test discovery
- Update CI/CD pipeline configurations to run tests from specific directories based on change scope (e.g., only run unit tests for small changes)
- Create template files or code snippets for common test file structures to ensure consistency
- Document the test organization structure in the project README and contributing guidelines
- Consider implementing a pre-commit hook that validates test file names and locations

## Continuation Context


Verify commands:
- find tests -name '*.test.ts' | wc -l
- grep -r 'describe\|test\|it' tests/ --include='*.test.ts' | head -n 5
- ls -la tests/unit tests/integration tests/e2e 2>/dev/null || echo 'Test directories not found'

Accept when:
- All test files use the .test.ts extension and are discoverable by the configured test runner
- Test files are organized into appropriate directories (unit/integration/e2e) based on their scope
- CI/CD pipeline successfully discovers and executes all tests without manual configuration for new test files
- Test coverage reports accurately reflect all test files in the project

## Enforcement

- Verified by: Automated CI/CD pipeline checks that validate test file naming patterns
- Verified by: Pre-commit hooks that verify test file locations and extensions
- Verified by: Code review checklist items for new test file additions
- Verified by: Periodic automated audits of test directory structure
- Violation handling: CI/CD pipeline fails if test files don't match expected naming patterns
- Violation handling: Pre-commit hooks block commits with incorrectly named or located test files
- Violation handling: Code review process flags violations for correction before merge
- Violation handling: Quarterly audits identify and track migration of non-compliant test files
- Exception process: Submit exception request to tech lead with justification and impact analysis
- Exception process: Document approved exceptions in ADR exceptions log with expiration date
- Exception process: Include migration plan for temporary exceptions
- Exception process: Review all active exceptions quarterly to assess if they can be resolved