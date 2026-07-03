# Adopt Async/Await Concurrency Model for External API Interactions: Functions Making Multiple

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all external API interactions and public-facing API implementations within the system.

## Context

- The codebase demonstrates consistent use of async/await patterns across multiple modules handling external API interactions and public-facing interfaces
- Pattern detected in 3 files (GitignoreUtils.ts, AugmentCodeAgent.ts, harness.ts) with 90% confidence, indicating a deliberate architectural choice
- Modern JavaScript/TypeScript applications require non-blocking I/O operations to maintain responsiveness when interacting with external services
- The paradigm.concurrency_model facet indicates this is a fundamental architectural decision affecting how concurrent operations are structured
- External APIs introduce latency and unpredictability that necessitate asynchronous handling to prevent blocking the main execution thread

## Problem Statement

Without a standardized concurrency model for external API interactions, the codebase risks inconsistent error handling, blocking operations that degrade performance, callback hell that reduces maintainability, and unpredictable behavior when multiple API calls need coordination. A unified approach is needed to ensure all external API interactions are non-blocking, maintainable, and follow consistent patterns.

## Decision

1. SHOULD: Functions making multiple independent API calls SHOULD use Promise.all() or Promise.allSettled() to parallelize requests

## Policy Block

- SHOULD Functions making multiple independent API calls SHOULD use Promise.all() or Promise.allSettled() to parallelize requests

In scope:
- All HTTP/HTTPS requests to external services
- Database queries and transactions
- File system operations that may block (large files, network mounts)
- Third-party SDK calls that perform I/O
- WebSocket and streaming API interactions
- Public API endpoints exposed by this system

Out of scope:
- Pure computational functions with no I/O
- Synchronous utility functions (string manipulation, math operations)
- In-memory data structure operations
- Configuration object initialization

Exceptions:
- EXC-001: Legacy code integration where refactoring async boundaries would require extensive changes across module boundaries
- EXC-002: Performance-critical hot paths where async overhead is measured and documented to cause unacceptable latency

## Rationale

- Pattern signature 9fc89443136b4a952c8158001bf1005f detected across 3 files with 90% confidence indicates this is an established architectural pattern
- Async/await provides superior readability and maintainability compared to callback-based or Promise chain approaches, reducing cognitive load for developers
- Non-blocking I/O is essential for scalability in Node.js/TypeScript environments where a single thread handles multiple concurrent requests
- Consistent concurrency model reduces bugs related to race conditions, deadlocks, and improper error propagation in asynchronous code

## Consequences

Positive:
- Improved application responsiveness and throughput by preventing blocking operations on external API calls
- Enhanced code readability with linear async/await syntax compared to nested callbacks or Promise chains
- Better error handling with standard try/catch blocks that work naturally with async/await
- Easier testing with async test frameworks that natively support Promise-based assertions
- Consistent patterns across the codebase reduce onboarding time for new developers

Negative:
- Async functions always return Promises, which adds slight overhead for simple operations
- Developers must understand Promise semantics and async execution model to avoid common pitfalls
- Stack traces can be harder to debug in async code, though modern tooling has improved this
- Mixing async and sync code requires careful attention to execution order and error propagation boundaries

## Alternatives

- Use callback-based concurrency model with error-first callbacks (rejected)
  Rejected because: Callback-based patterns lead to callback hell, poor error handling, and reduced code maintainability. Modern JavaScript/TypeScript has moved away from this pattern.
  When valid: Only valid for legacy Node.js codebases (pre-ES2017) or when integrating with old libraries that don't support Promises
- Use raw Promises with .then()/.catch() chains (rejected)
  Rejected because: Promise chains are less readable than async/await and make error handling more complex. Async/await is syntactic sugar over Promises that improves developer experience.
  When valid: Acceptable for simple one-off Promise operations or when chaining transformations where async/await would be verbose
- Use reactive programming with RxJS Observables (rejected)
  Rejected because: RxJS adds significant complexity and learning curve. While powerful for complex event streams, it's overkill for standard API request/response patterns.
  When valid: Valid for complex event-driven scenarios with multiple data streams, real-time updates, or sophisticated operator compositions

## Risks

- Unhandled Promise rejections can crash Node.js applications or cause silent failures
  Mitigation: Implement global unhandled rejection handlers, enforce try/catch in linting rules, and use TypeScript strict mode to catch missing await keywords
  Owner: Engineering team
- Excessive parallelization with Promise.all() can overwhelm external APIs or exhaust connection pools
  Mitigation: Implement rate limiting, connection pooling, and use Promise batching utilities (e.g., p-limit) for controlled concurrency
  Owner: Platform team
- Async/await can hide performance issues by making sequential operations appear synchronous
  Mitigation: Code review guidelines emphasizing identification of parallelizable operations, performance profiling in CI/CD pipeline
  Owner: Engineering team

## Implementation Notes

- Use ESLint rules like @typescript-eslint/no-floating-promises and @typescript-eslint/require-await to enforce async patterns
- Wrap all external API client libraries in async adapter layers to ensure consistent interfaces across the codebase
- Document async function behavior in JSDoc comments, especially regarding error conditions and timeout behavior
- Use TypeScript's Promise<T> return type annotations explicitly to make async contracts clear to consumers
- Consider implementing retry logic and circuit breakers at the async wrapper level for resilient external API interactions

## Continuation Context


Verify commands:
- grep -r 'function.*(' src/ | grep -v 'async' | grep -E '(fetch|axios|http|request)' || echo 'No synchronous API calls found'
- eslint --rule '@typescript-eslint/no-floating-promises: error' src/
- grep -r '\.then(' src/ --include='*.ts' | wc -l | awk '{if($1>10) print "Warning: Found " $1 " .then() usages, prefer async/await"; else print "OK"}'

Accept when:
- All functions making external API calls are declared with async keyword and return Promise types
- ESLint checks pass with no-floating-promises and require-await rules enabled
- Code review confirms proper try/catch error handling around all await statements for external APIs
- No synchronous blocking operations detected in API interaction code paths

## Enforcement

- Verified by: ESLint rules in CI/CD pipeline enforcing async/await patterns and catching floating promises
- Verified by: TypeScript compiler strict mode catching missing await keywords on Promise-returning functions
- Verified by: Code review checklist requiring verification of async patterns in all API-related pull requests
- Verified by: Automated static analysis scanning for synchronous API calls in pre-commit hooks
- Violation handling: CI/CD pipeline fails if ESLint rules detect violations of async/await patterns
- Violation handling: Pull requests blocked until code review confirms compliance with async patterns
- Violation handling: Runtime monitoring alerts on unhandled Promise rejections in production
- Violation handling: Quarterly architecture reviews identify and prioritize remediation of legacy synchronous code
- Exception process: Developer submits exception request with technical justification and impact analysis
- Exception process: Tech lead reviews and approves/rejects based on policy exception criteria (EXC-001, EXC-002)
- Exception process: Approved exceptions documented in code with EXCEPTION tag and linked to tracking ticket
- Exception process: Exception registry maintained and reviewed quarterly for potential remediation