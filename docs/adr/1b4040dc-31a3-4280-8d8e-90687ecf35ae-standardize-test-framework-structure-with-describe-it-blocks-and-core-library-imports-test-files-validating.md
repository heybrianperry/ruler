# Standardize Test Framework Structure with describe/it Blocks and Core Library Imports: Test Files Validating

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains 37 test files exhibiting consistent structural patterns using describe/it test organization blocks
- Test files systematically import core Node.js libraries including 'fs/promises', 'path', 'os', and 'child_process' for filesystem and environment operations
- Tests validate configuration files (mcp.json, ruler.toml, settings.json) through JSON.parse operations and file system assertions
- Integration and unit tests span multiple agent implementations (GooseAgent, JetBrainsAiAssistantAgent, AntigravityAgent) and CLI workflows
- Test harness utilities are consistently imported from relative paths ('./harness', '../tests/harness') indicating shared test infrastructure

## Problem Statement

Without a standardized testing strategy, test files may diverge in structure, library usage, and organization patterns, leading to inconsistent test maintenance, reduced readability, and difficulty in establishing reliable verification workflows across agent implementations and CLI operations.

## Decision

1. MUST: Test files validating JSON configuration MUST use JSON.parse for input validation and content verification

## Policy Block

- MUST Test files validating JSON configuration MUST use JSON.parse for input validation and content verification

In scope:
- All test files in tests/ directory including unit, integration, and CLI test suites
- Test files validating agent implementations (GooseAgent, JetBrainsAiAssistantAgent, AntigravityAgent, etc.)
- Test files verifying configuration file handling (mcp.json, ruler.toml, settings.json)
- Test harness utilities and shared test infrastructure modules

Out of scope:
- Source code files outside the tests/ directory
- Build scripts and deployment configurations
- Documentation and example files
- Third-party library test suites

## Rationale

- Pattern detected across 37 test files with 92.59% confidence indicates strong architectural consistency in testing approach
- Consistent use of describe/it blocks enables predictable test structure and facilitates test runner integration
- Systematic import of core Node.js libraries ('fs/promises', 'path', 'os') reflects filesystem-centric testing requirements for configuration validation
- Shared test harness pattern reduces duplication and establishes reusable test infrastructure for project setup and teardown

## Consequences

Positive:
- Uniform test structure improves readability and maintainability across 37+ test files
- Consistent library imports establish clear dependencies for filesystem and environment testing operations
- Shared test harness utilities reduce code duplication and enable standardized test isolation patterns
- Predictable test organization facilitates onboarding and test discovery for new contributors

Negative:
- Rigid test structure may constrain alternative testing approaches or specialized test patterns
- Dependency on specific core Node.js libraries couples tests to Node.js runtime environment
- Shared test harness creates central point of failure if harness utilities contain defects
- Consistent patterns may obscure legitimate variations in testing requirements across different component types

## Alternatives

- Use flat test structure without describe/it nesting (rejected)
  Rejected because: Flat structure lacks hierarchical organization needed for complex test suites spanning multiple agents and CLI workflows
  When valid: May be appropriate for simple single-function unit tests with no setup/teardown requirements
- Inline test utilities without shared harness (rejected)
  Rejected because: Inline utilities would duplicate setupTestProject/teardownTestProject logic across 37+ test files
  When valid: Acceptable for isolated tests with unique setup requirements not shared by other test files
- Use test framework-specific assertion libraries instead of core Node.js imports (deferred)
  Rejected because: Not rejected; current pattern uses core libraries but could be enhanced with framework-specific utilities
  When valid: Valid when test framework provides superior filesystem mocking or assertion capabilities

## Risks

- Test harness changes may break multiple test files simultaneously due to shared dependency
  Mitigation: Implement versioning or compatibility layer for test harness utilities; maintain comprehensive harness test coverage
  Owner: engineering team
- Filesystem-dependent tests may exhibit platform-specific behavior across Windows, macOS, and Linux
  Mitigation: Use path.join and os.tmpdir() consistently; validate tests on multiple platforms in CI pipeline
  Owner: engineering team
- JSON.parse operations on malformed test fixtures may produce unclear error messages
  Mitigation: Wrap JSON.parse in try-catch blocks with descriptive error context; validate test fixtures in pre-commit hooks
  Owner: engineering team

## Implementation Notes

- Import test harness utilities from './harness' or '../tests/harness' depending on test file location relative to tests/ directory
- Use setupTestProject with file structure object to create isolated test environments with required configuration files
- Call teardownTestProject in afterEach hooks to ensure cleanup even when tests fail
- Organize describe blocks by component name (e.g., 'GooseAgent', 'CLI nested toggle precedence') and nest it blocks for specific behaviors
- Use JSON.stringify with null and 2 parameters for consistent formatting when creating test fixture JSON files

## Continuation Context


Verify commands:
- grep -r "describe(" tests/ | wc -l
- grep -r "from 'fs/promises'\|from 'path'\|from 'os'" tests/ | wc -l
- grep -r "JSON.parse" tests/ | wc -l
- find tests/ -name '*.test.ts' -exec grep -l "./harness\|../tests/harness" {} \; | wc -l

Accept when:
- All test files in tests/ directory use describe/it block structure for test organization
- Test files performing filesystem operations import at least one of 'fs/promises', 'path', or 'os'
- Test files validating JSON configuration use JSON.parse for content verification
- Integration tests use shared test harness utilities for project setup and teardown

## Enforcement

- Verified by: Code review checklist verifying test structure compliance
- Verified by: CI pipeline grep-based verification of describe/it patterns and core library imports
- Verified by: Automated linting rules detecting missing test harness imports in integration tests
- Violation handling: CI pipeline fails if test files lack describe/it structure or required core library imports
- Violation handling: Code review feedback requests restructuring of non-compliant test files
- Violation handling: Test coverage reports flag files not using shared harness utilities
- Exception process: Document justification for alternative test structure in test file header comment
- Exception process: Obtain approval from testing infrastructure maintainer for harness-free tests
- Exception process: Add exception marker comment (e.g., // ADR-AUTO-EXCEPTION: specialized test pattern) for automated tooling