# Adopt Async/Await Pattern for External API Integration: Synchronous Blocking Operations

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all external API integration code and agent implementations that interact with public/external APIs.

## Context

- The codebase contains multiple agent implementations (CursorAgent, AgentsMdAgent, AiderAgent, AugmentCodeAgent) that interact with external APIs and perform I/O operations
- Asynchronous operations are prevalent across 11 files with 90% significance, indicating a consistent architectural pattern for handling concurrent operations
- External API calls require non-blocking I/O to maintain application responsiveness and enable efficient resource utilization
- The paradigm.concurrency_model facet indicates this is a fundamental architectural decision about how concurrent operations are structured
- Modern TypeScript/JavaScript environments provide native async/await support, making it the idiomatic choice for asynchronous programming

## Problem Statement

External API integrations and I/O-bound operations can block the main execution thread, leading to poor application performance, unresponsive user interfaces, and inefficient resource utilization. Without a consistent concurrency model, different parts of the codebase may handle asynchronous operations inconsistently, creating maintenance challenges and potential race conditions.

## Decision

1. MUST_NOT: Synchronous blocking operations MUST NOT be used for external API calls or network I/O

## Policy Block

- MUST_NOT Synchronous blocking operations MUST NOT be used for external API calls or network I/O

In scope:
- All agent implementations (CursorAgent, AgentsMdAgent, AiderAgent, AugmentCodeAgent)
- External API integration code in lib.ts and similar modules
- File system operations in utility modules (GitignoreUtils, SkillsUtils, SkillsProcessor)
- Test harness code that interacts with external systems
- Revert engine and state management operations involving I/O

Out of scope:
- Pure computational functions with no I/O operations
- Synchronous utility functions for data transformation
- Type definitions and interface declarations
- Configuration constants and static data structures

Exceptions:
- EX-001: Legacy third-party libraries that only provide callback-based APIs
- EX-002: Performance-critical hot paths where async overhead is measurably significant

## Rationale

- Pattern detected across 11 files with 90% confidence indicates this is an established architectural standard in the codebase
- Async/await provides superior readability and maintainability compared to callback-based or Promise-chaining approaches
- Non-blocking I/O is essential for maintaining responsiveness in agent-based systems that interact with multiple external services
- TypeScript's native support for async/await enables strong type checking and compile-time validation of asynchronous code flows

## Consequences

Positive:
- Improved application responsiveness by preventing blocking operations on the main thread
- Consistent concurrency model across all agent implementations reduces cognitive load for developers
- Better error handling and stack traces compared to callback-based approaches
- Enables efficient parallel execution of independent API calls using Promise.all()
- Facilitates testing with async test frameworks and mock async operations

Negative:
- Async/await introduces slight performance overhead compared to synchronous operations for trivial tasks
- Requires developers to understand Promise semantics and async execution context
- Debugging async code can be more complex due to asynchronous stack traces
- May complicate integration with legacy synchronous APIs requiring wrapper code

## Alternatives

- Callback-based asynchronous pattern using Node.js-style callbacks (rejected)
  Rejected because: Leads to callback hell, poor error handling, and reduced code readability. Async/await is the modern standard with better developer experience.
  When valid: Only when integrating with legacy libraries that exclusively provide callback APIs
- Promise chaining with .then() and .catch() (rejected)
  Rejected because: While functional, Promise chaining is more verbose and harder to read than async/await syntax. Async/await is syntactic sugar over Promises with better ergonomics.
  When valid: Acceptable for simple single-promise operations where async function declaration adds unnecessary overhead
- Synchronous blocking operations with worker threads for I/O (rejected)
  Rejected because: Adds significant complexity with worker thread management, message passing overhead, and serialization costs. Async/await provides simpler non-blocking I/O.
  When valid: Only for CPU-intensive computational tasks, not for I/O-bound operations

## Risks

- Unhandled promise rejections causing silent failures or application crashes
  Mitigation: Enforce try/catch blocks in all async functions, enable Node.js unhandledRejection warnings, implement global error handlers
  Owner: Engineering team
- Race conditions when multiple async operations access shared state without proper synchronization
  Mitigation: Use mutex/semaphore patterns for critical sections, implement atomic operations, conduct code reviews focusing on concurrent access patterns
  Owner: Engineering team
- Memory leaks from unclosed resources or hanging promises in long-running async operations
  Mitigation: Implement timeout mechanisms, use AbortController for cancellable operations, monitor memory usage in production
  Owner: DevOps and Engineering team

## Implementation Notes

- Use TypeScript's strict mode to ensure all async functions have proper return type annotations (Promise<T>)
- Implement a standard timeout wrapper utility for all external API calls (e.g., withTimeout(promise, ms))
- For parallel operations, prefer Promise.allSettled() over Promise.all() when partial failures should not abort the entire operation
- Document async function behavior in JSDoc comments, especially regarding error handling and timeout behavior
- Use ESLint rules like 'no-floating-promises' and 'require-await' to catch common async/await mistakes

## Continuation Context


Verify commands:
- grep -r 'async function\|async (' src/ --include='*.ts' | wc -l
- grep -r '\.then(' src/ --include='*.ts' | grep -v 'Promise.all\|Promise.allSettled' | wc -l
- npx eslint src/ --rule 'require-await: error' --rule '@typescript-eslint/no-floating-promises: error'

Accept when:
- All external API calls in agent implementations use async/await syntax
- ESLint checks pass with no-floating-promises and require-await rules enabled
- Ratio of async/await usage to .then() chaining is >80% in API integration code
- All async functions include error handling (try/catch or .catch())

## Enforcement

- Verified by: ESLint rules enforced in CI pipeline (no-floating-promises, require-await, @typescript-eslint/promise-function-async)
- Verified by: Code review checklist includes verification of async/await usage for API calls
- Verified by: TypeScript compiler strict mode catches missing Promise return types
- Verified by: Automated tests verify async behavior and error handling
- Violation handling: CI build fails if ESLint async/await rules are violated
- Violation handling: Code review blocks merge if synchronous blocking operations are used for external APIs
- Violation handling: Pull requests must include test coverage for async error handling paths
- Violation handling: Violations in existing code trigger technical debt tickets for refactoring
- Exception process: Developer submits exception request with justification to tech lead
- Exception process: For legacy API integration: provide migration plan and timeline for async wrapper implementation
- Exception process: For performance-critical paths: provide benchmark data and profiling results
- Exception process: Approved exceptions must be documented in code comments with ADR reference and expiration date