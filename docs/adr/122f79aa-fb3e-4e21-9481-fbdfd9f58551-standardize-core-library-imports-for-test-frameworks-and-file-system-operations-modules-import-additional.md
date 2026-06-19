# Standardize Core Library Imports for Test Frameworks and File System Operations: Modules Import Additional

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all agent implementations and test suites within the project.

## Context

- The codebase demonstrates consistent usage of Node.js core libraries (fs/promises, path, os) across 45 files with 91.72% confidence, indicating an established architectural pattern.
- Test files systematically import describe/it/expect from test frameworks alongside fs/promises and path modules, suggesting a standardized testing and file system interaction approach.
- Agent implementations (AmazonQCliAgent, WindsurfAgent, AmpAgent, JunieAgent) consistently import core modules and utility classes, establishing a common dependency pattern.
- The pattern spans both test infrastructure (tests/apply-mcp.firebase.test.ts, tests/mcp-backup-prevention.test.ts) and production agent code (src/agents/*.ts), indicating project-wide adoption.
- JSON parsing operations consistently follow file system reads, suggesting a standardized configuration loading pattern across the codebase.

## Problem Statement

Without standardized core library imports and consistent module usage patterns, the codebase risks fragmentation in how file system operations, path manipulations, and test framework integrations are implemented. This can lead to inconsistent error handling, reduced code maintainability, and difficulty in enforcing security practices like input validation across agent implementations and test suites.

## Decision

1. MAY: Modules MAY import additional Node.js core libraries (os, child_process) when required for specific functionality.

## Policy Block

- MAY Modules MAY import additional Node.js core libraries (os, child_process) when required for specific functionality.

In scope:
- All TypeScript files in src/agents/ directory
- All test files in tests/ directory and subdirectories
- Core utility modules in src/core/
- MCP integration modules in src/mcp/
- CLI implementation files requiring file system access

Out of scope:
- Third-party library code in node_modules/
- Generated code or build artifacts
- Documentation files and markdown content
- Configuration files (package.json, tsconfig.json) that do not contain executable code

Exceptions:
- EXC-001: Legacy code modules require synchronous fs operations for compatibility with existing synchronous APIs
- EXC-002: Performance-critical paths require synchronous file operations to avoid async overhead

## Rationale

- The fs/promises API provides consistent async/await patterns that align with modern TypeScript/JavaScript practices and improve error handling compared to callback-based approaches.
- Centralized utility imports (FileSystemUtils, IAgent interface) reduce code duplication and ensure consistent behavior across 45+ files, as evidenced by the detected pattern.
- Standardized test framework imports enable consistent test structure and maintainability across the test suite, supporting the observed describe/it/expect pattern in multiple test files.
- The pattern's 91.72% confidence across 45 files indicates organic adoption and validates this as an established architectural convention worth formalizing.

## Consequences

Positive:
- Consistent async/await patterns improve code readability and reduce callback complexity across file system operations.
- Centralized utility modules reduce code duplication and simplify maintenance when updating common functionality.
- Standardized imports enable easier onboarding for new developers and clearer code review expectations.
- Cross-platform compatibility is ensured through consistent use of the path module for file path operations.

Negative:
- Async/await patterns may introduce slight performance overhead compared to synchronous operations in non-critical paths.
- Strict import requirements may require refactoring of existing code that uses alternative patterns.
- Dependency on specific utility modules creates coupling that must be managed during refactoring.
- Additional abstraction layers (AbstractAgent, AgentsMdAgent) may obscure implementation details for developers unfamiliar with the inheritance hierarchy.

## Alternatives

- Use synchronous fs methods (fs.readFileSync, fs.writeFileSync) throughout the codebase for simplicity (rejected)
  Rejected because: Synchronous operations block the event loop and prevent concurrent operations, reducing scalability and responsiveness in CLI and agent operations
  When valid: Only valid for small utility scripts or initialization code that runs once before async operations begin
- Allow mixed usage of fs/promises, callback-based fs, and synchronous fs methods based on developer preference (rejected)
  Rejected because: Mixed patterns create inconsistent error handling, complicate code reviews, and make it difficult to enforce security practices like input validation
  When valid: Not recommended; consistency provides more value than flexibility in this context
- Create a custom file system abstraction layer that wraps all fs operations (deferred)
  Rejected because: FileSystemUtils already provides targeted utilities; a complete abstraction layer may be over-engineering without clear benefits
  When valid: Consider if cross-platform compatibility issues arise or if mocking/testing requirements become more complex

## Risks

- Existing code using callback-based or synchronous fs methods may require significant refactoring effort
  Mitigation: Implement gradual migration strategy with linting rules that warn on deprecated patterns; prioritize high-traffic code paths first
  Owner: Engineering team
- Developers unfamiliar with async/await patterns may introduce error handling bugs or unhandled promise rejections
  Mitigation: Provide code examples and documentation; implement ESLint rules to catch common async/await mistakes; require code review for async operations
  Owner: Engineering team and DevOps
- Tight coupling to utility modules (FileSystemUtils, IAgent) may complicate future refactoring or module reorganization
  Mitigation: Document module boundaries and responsibilities; use dependency injection where appropriate; maintain clear interfaces between modules
  Owner: Architecture team

## Implementation Notes

- Use ESLint rules to enforce fs/promises imports and flag usage of deprecated fs callback methods or synchronous operations.
- Create code snippets or templates for common patterns (read JSON config, write file with backup, path joining) to accelerate adoption.
- Document the FileSystemUtils module API and provide examples of when to use utility functions versus direct fs/promises calls.
- Establish test harness patterns in ./harness module and document standard setup/teardown patterns for file system tests.
- For agent implementations, provide a base class checklist documenting which methods to override and which utilities to import.

## Continuation Context


Verify commands:
- grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v 'fs/promises' | wc -l
- grep -r "from 'fs'" src/ tests/ --include='*.ts' | grep -v "from 'fs/promises'" | wc -l
- npx eslint src/ tests/ --rule 'no-restricted-imports: [error, {paths: [{name: fs, message: Use fs/promises instead}]}]'
- grep -r "JSON.parse" src/ tests/ --include='*.ts' -A 2 -B 2 | grep -c 'try\|catch'

Accept when:
- All grep commands for non-promises fs imports return zero matches in src/ and tests/ directories
- ESLint validation passes with no restricted import violations for fs module
- At least 90% of JSON.parse operations are wrapped in try-catch blocks or use error handling utilities
- All new agent implementations extend AbstractAgent or AgentsMdAgent base classes
- All test files use standardized test framework imports (describe, it, expect) from the configured test runner

## Enforcement

- Verified by: ESLint rules enforcing fs/promises imports and flagging deprecated fs usage patterns
- Verified by: Code review checklist requiring verification of async/await patterns and error handling
- Verified by: Automated CI pipeline running grep-based verification commands to detect non-compliant imports
- Verified by: TypeScript compiler checks ensuring proper module imports and type safety
- Violation handling: CI pipeline fails on ESLint violations related to restricted fs imports
- Violation handling: Code review process blocks merge requests that introduce callback-based or synchronous fs operations without documented exceptions
- Violation handling: Automated comments on pull requests flag potential violations with links to this ADR and remediation guidance
- Violation handling: Quarterly code audits identify and prioritize refactoring of legacy code that violates these standards
- Exception process: Developer documents technical justification in code comments and links to this ADR
- Exception process: Exception request submitted via architecture review process with performance data or compatibility requirements
- Exception process: Team lead or architect reviews exception request and approves/rejects with documented rationale
- Exception process: Approved exceptions tracked in technical debt backlog with migration plan and timeline