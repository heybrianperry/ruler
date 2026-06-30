# Adopt Async Function Naming Convention for Test Setup Utilities: Test Setup Utilities

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Test files across the codebase use describe/it test frameworks with async setup utilities for file system operations
- Test harness functions like readGeminiSettings, pathExists, writeProject, writeRulerGitignore, writeAgent, writeNestedAgent, and writeNestedProjectConfig perform I/O operations that require asynchronous execution
- Integration and unit tests require consistent file system state management including reading JSON configuration files, creating test directories, and cleaning up resources
- The paradigm.concurrency_model facet captures async function naming patterns that signal asynchronous behavior in test setup and teardown operations
- Five test files demonstrate consistent use of async utilities for managing test fixtures, configuration files, and temporary file system state

## Problem Statement

Test setup and teardown utilities that perform file system operations need clear naming conventions to signal their asynchronous nature and ensure proper await handling in test code, preventing race conditions and test flakiness caused by unresolved promises.

## Decision

1. MUST: Test setup utilities that perform file system operations (read, write, exists checks) MUST be implemented as async functions

## Policy Block

- MUST Test setup utilities that perform file system operations (read, write, exists checks) MUST be implemented as async functions

## Rationale

- The evidence shows 5 test files consistently using async utilities (readGeminiSettings, pathExists, writeProject, writeRulerGitignore, writeAgent, writeNestedAgent, writeNestedProjectConfig) for file system operations, indicating an established pattern
- JSON parsing operations (JSON.parse(await fs.readFile(...))) appear in multiple test files, demonstrating the need for async handling when reading configuration files
- The paradigm.concurrency_model facet with 92.02% confidence across 5 files indicates this is a deliberate architectural choice rather than isolated implementation
- Consistent verb-based naming (read*, write*, path*) provides clear semantic signals about the asynchronous nature of test utilities

## Consequences

Positive:
- Clear naming conventions reduce cognitive load and make asynchronous behavior explicit in test code
- Consistent async/await patterns prevent race conditions and test flakiness caused by unresolved promises
- Reusable test utilities in harness modules promote DRY principles and reduce duplication across test files
- Explicit async function signatures enable TypeScript/IDE tooling to enforce proper await usage at compile time

Negative:
- Async test utilities add complexity compared to synchronous alternatives, requiring developers to understand promise semantics
- Test execution time may increase due to sequential async operations if not properly parallelized
- Debugging async test failures can be more challenging due to promise chain stack traces

## Alternatives

- Use synchronous file system operations (fs.readFileSync, fs.writeFileSync) for test utilities (rejected)
  Rejected because: Synchronous operations block the event loop and prevent parallel test execution, reducing test suite performance and limiting scalability
  When valid: Only valid for small test suites with minimal file I/O where performance is not a concern
- Use callback-based file system APIs with promisify wrappers (rejected)
  Rejected because: Callback-based APIs are less readable and harder to maintain compared to native async/await syntax, and the evidence shows direct use of fs/promises module
  When valid: Valid for legacy codebases that have not migrated to Promise-based APIs
- Inline all file system operations directly in test cases without utility functions (rejected)
  Rejected because: Inlining creates duplication across test files and makes it harder to maintain consistent setup patterns, as evidenced by the harness module usage
  When valid: Valid for one-off test cases with unique setup requirements that are not reusable

## Risks

- Developers may forget to await async test utilities, causing intermittent test failures that are difficult to diagnose
  Mitigation: Enable ESLint rules like @typescript-eslint/no-floating-promises to catch missing await statements at lint time
  Owner: Engineering team
- Sequential async operations in test setup may create performance bottlenecks as test suites grow
  Mitigation: Use Promise.all() to parallelize independent file system operations where possible, and document patterns for parallel test setup
  Owner: Engineering team
- Inconsistent error handling in async utilities may cause test failures to be masked or reported incorrectly
  Mitigation: Establish standard error handling patterns for test utilities and ensure all async functions properly propagate errors
  Owner: Engineering team

## Implementation Notes

- Import fs/promises module rather than callback-based fs module for all test utilities that perform file system operations
- Export async test utilities from a centralized test harness module (e.g., tests/harness) to promote reuse across test files
- Use descriptive verb-based names (read*, write*, create*, delete*, exists*) that clearly indicate the I/O operation being performed
- Combine related operations (e.g., readFile + JSON.parse) into single async utilities to reduce boilerplate in test files

## Continuation Context


Verify commands:
- grep -r "async function" tests/ | grep -E "(read|write|path|create|delete)" | wc -l
- grep -r "from 'fs/promises'" tests/ | wc -l
- grep -r "await.*\(read\|write\|path\)" tests/ | wc -l

Accept when:
- All test utilities that perform file system operations are declared as async functions
- Test files consistently import fs/promises module and await all async utility calls
- Async test utilities use verb-based naming conventions that clearly indicate I/O operations

## Enforcement

- Verified by: ESLint rules (@typescript-eslint/no-floating-promises, @typescript-eslint/require-await) enforced in CI pipeline
- Verified by: Code review checklist includes verification of async/await patterns in test utilities
- Verified by: TypeScript compiler strict mode catches missing await on Promise-returning functions
- Violation handling: CI build fails if ESLint detects floating promises or missing await statements in test files
- Violation handling: Code review blocks merge if async test utilities do not follow naming conventions
- Violation handling: Test failures caused by unresolved promises are investigated and fixed with priority
- Exception process: Exceptions for synchronous file operations must be justified in code comments with performance or compatibility rationale
- Exception process: Exceptions require approval from tech lead during code review
- Exception process: All exceptions must be documented in test file headers with expiration date for review