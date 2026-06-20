# Adopt TypeScript Test Files with .test.ts Extension as Standard Testing Convention: Typescript Test Files

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all TypeScript test file creation and organization within the codebase.

## Context

- The codebase contains 121 test files following a consistent naming pattern with .test.ts extension, indicating a well-established testing convention across the project
- Test files are organized into distinct directories (tests/e2e, tests/integration, tests/unit) demonstrating a clear separation of test types and concerns
- The pattern shows comprehensive test coverage across multiple layers including unit tests for agents, integration tests for configuration and CLI behavior, and end-to-end tests for complete workflows
- TypeScript is the primary language for both application code and test code, requiring consistent file extensions and tooling configuration for test discovery and execution
- The testing infrastructure needs to automatically discover and execute tests as part of CI/CD pipelines without manual configuration of individual test files

## Problem Statement

Without a standardized test file naming convention, test discovery becomes unreliable, CI/CD pipelines may miss test files, developers cannot easily distinguish test files from implementation files, and tooling configuration becomes fragmented. The codebase needs a consistent, automatable approach to identify and execute all test files across different test types (unit, integration, e2e) while maintaining clear separation from production code.

## Decision

1. MUST: All TypeScript test files MUST use the .test.ts file extension

## Policy Block

- MUST All TypeScript test files MUST use the .test.ts file extension

In scope:
- All TypeScript test files for unit, integration, and end-to-end testing
- Test discovery configuration in test runners (Jest, Vitest, Mocha, etc.)
- CI/CD pipeline test execution steps
- IDE and editor test runner integrations
- Code coverage reporting tools

Out of scope:
- JavaScript test files (.test.js) in legacy codebases
- Test files in other languages (Python, Go, etc.)
- Specification files using .spec.ts extension (if used in parallel)
- Test fixtures, mocks, or helper files that are not executable tests
- Documentation or example files that demonstrate testing patterns

Exceptions:
- EXC-001: Legacy test files exist with .spec.ts extension from previous conventions
- EXC-002: Third-party or vendor test files require different naming conventions

## Rationale

- Pattern detected across 121 files with 91.92% confidence indicates this is a mature, well-adopted convention that has proven effective in practice
- Consistent .test.ts extension enables simple glob patterns (e.g., **/*.test.ts) for test discovery, reducing configuration complexity and improving CI/CD reliability
- Separation into tests/ directory with type-based subdirectories provides clear organization and allows selective test execution (e.g., running only unit tests for fast feedback)
- TypeScript test files benefit from type checking and IDE support, catching errors in test code before execution and improving test maintainability

## Consequences

Positive:
- Automated test discovery works reliably across all CI/CD tools and test runners without manual configuration
- Developers can instantly identify test files versus production code through consistent naming convention
- Test organization by type (unit/integration/e2e) enables selective test execution for faster feedback loops
- IDE and editor tooling can automatically recognize and provide specialized support for test files
- New team members can quickly understand the testing structure and locate relevant tests

Negative:
- Existing codebases using .spec.ts or other conventions require migration effort to standardize
- Strict directory structure may feel constraining for small projects or prototypes
- Test files are physically separated from implementation files, requiring navigation between directories
- Tooling configuration must be updated if migrating from a different test file naming convention

## Alternatives

- Use .spec.ts extension following Angular and other framework conventions (rejected)
  Rejected because: The codebase has already standardized on .test.ts with 121 files, and changing would require significant migration effort without clear benefit. The .test.ts convention is equally well-supported by tooling.
  When valid: Valid for new projects without existing test files or when integrating with Angular-specific tooling that expects .spec.ts
- Co-locate test files with source files using __tests__/ subdirectories (rejected)
  Rejected because: Centralized tests/ directory provides clearer separation and easier test suite management. Co-location can clutter source directories and complicate build configurations.
  When valid: Valid for component-based architectures where tests are tightly coupled to specific components and rarely shared
- Use mixed naming conventions (.test.ts and .spec.ts) for different test types (rejected)
  Rejected because: Multiple conventions increase cognitive load, complicate test discovery patterns, and create inconsistency. Single convention is simpler and equally expressive.
  When valid: Never recommended - consistency is more valuable than semantic distinction through file extensions

## Risks

- Test files may be accidentally excluded from version control or deployment if .test.ts pattern is not properly configured in .gitignore or build tools
  Mitigation: Explicitly include tests/ directory in version control and document test file patterns in CI/CD configuration. Add pre-commit hooks to verify test files are tracked.
  Owner: DevOps and Engineering Team
- Developers may create test files with incorrect naming or location, causing tests to be silently skipped
  Mitigation: Implement linting rules or pre-commit hooks that verify test file naming conventions. Add CI checks that fail if test files are found outside expected directories.
  Owner: Engineering Team
- Migration from existing test naming conventions may introduce errors or miss test files
  Mitigation: Create automated migration scripts with verification steps. Run parallel test execution with old and new patterns during transition period. Maintain checklist of all test files to verify complete migration.
  Owner: Tech Lead and Engineering Team

## Implementation Notes

- Configure test runner (Jest, Vitest, etc.) with testMatch pattern: ['**/*.test.ts'] or similar to automatically discover all test files
- Update tsconfig.json to include tests/ directory and ensure test files are type-checked alongside production code
- Add ESLint rules or custom linting to enforce test file naming conventions and directory structure
- Document the testing structure in CONTRIBUTING.md or TESTING.md with examples of proper test file creation and organization
- Set up IDE workspace settings to recognize .test.ts files and provide appropriate test runner integrations

## Continuation Context


Verify commands:
- find tests/ -type f -name '*.ts' ! -name '*.test.ts' | grep -v 'helpers\|fixtures\|mocks' && echo 'Found non-test TypeScript files in tests/' || echo 'All test files follow .test.ts convention'
- grep -r 'testMatch\|testRegex' jest.config.* tsconfig.json package.json | grep -q '\.test\.ts' && echo 'Test runner configured for .test.ts pattern' || echo 'Warning: Test runner may not be configured correctly'
- ls tests/unit/ tests/integration/ tests/e2e/ 2>/dev/null && echo 'Test directory structure exists' || echo 'Warning: Expected test directories not found'

Accept when:
- All TypeScript test files in the codebase use the .test.ts extension and are located under the tests/ directory
- Test runner configuration includes glob patterns that match **/*.test.ts files
- CI/CD pipeline successfully discovers and executes all test files without manual configuration of individual test paths
- No TypeScript test files exist in production source directories outside of tests/

## Enforcement

- Verified by: Automated CI/CD pipeline checks that verify test file naming conventions
- Verified by: Pre-commit hooks that validate test file locations and naming patterns
- Verified by: Code review checklist items for new test file creation
- Verified by: Periodic audits using find and grep commands to identify non-compliant test files
- Violation handling: CI pipeline fails if test files are found with incorrect naming or location
- Violation handling: Pre-commit hooks block commits containing improperly named test files
- Violation handling: Code review process flags violations and requests corrections before merge
- Violation handling: Automated reports generated weekly identifying any non-compliant test files for remediation
- Exception process: Developer submits exception request to tech lead with justification for alternative naming
- Exception process: Tech lead reviews request and evaluates impact on test discovery and CI/CD
- Exception process: If approved, exception is documented in project README.md or TESTING.md with rationale
- Exception process: Exception is reviewed quarterly to determine if it can be eliminated through refactoring