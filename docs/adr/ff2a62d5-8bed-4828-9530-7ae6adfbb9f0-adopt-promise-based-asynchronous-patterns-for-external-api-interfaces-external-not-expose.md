# Adopt Promise-Based Asynchronous Patterns for External API Interfaces: External Not Expose

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all external API interfaces, agent implementations, and public-facing modules. All new external APIs MUST follow the promise-based asynchronous concurrency model defined herein.

## Context

- The codebase contains multiple agent implementations (CursorAgent, CopilotAgent, AiderAgent, AgentsMdAgent, AugmentCodeAgent) that expose external APIs requiring consistent concurrency handling
- Core utilities (SkillsUtils, SkillsProcessor, GitignoreUtils) and engine components (revert-engine) provide public interfaces that must handle asynchronous operations reliably
- Pattern signature 9fc89443136b4a952c8158001bf1005f detected across 12 files with 90% confidence indicates a standardized approach to concurrency management
- The facet 'paradigm.concurrency_model' suggests this pattern specifically addresses how concurrent operations are structured and exposed in public APIs
- TypeScript/JavaScript ecosystem best practices favor promise-based async patterns over callbacks or synchronous blocking operations for I/O-bound tasks

## Problem Statement

External APIs and agent interfaces require a consistent, predictable concurrency model that enables non-blocking operations, proper error propagation, and composable asynchronous workflows. Without a standardized approach, different modules may implement incompatible concurrency patterns, leading to integration difficulties, error handling inconsistencies, and reduced code maintainability across the agent ecosystem.

## Decision

1. MUST_NOT: External APIs MUST NOT expose callback-based interfaces for asynchronous operations

## Policy Block

- MUST_NOT External APIs MUST NOT expose callback-based interfaces for asynchronous operations

In scope:
- All agent implementations (CursorAgent, CopilotAgent, AiderAgent, AgentsMdAgent, AugmentCodeAgent)
- Core utility modules (SkillsUtils, SkillsProcessor, GitignoreUtils)
- Engine components (revert-engine, core processing modules)
- Public-facing library interfaces (lib.ts exports)
- Test harness and integration test utilities

Out of scope:
- Internal synchronous helper functions not exposed in public APIs
- Pure computational functions with no I/O or side effects
- Type definitions and interface declarations
- Configuration objects and constants

Exceptions:
- EXC-001: Performance-critical hot paths where synchronous operations are demonstrably faster and blocking is acceptable
- EXC-002: Interfacing with legacy third-party libraries that only provide callback-based APIs

## Rationale

- Pattern detected with 90% confidence across 12 files indicates this is an established architectural standard in the codebase
- Promise-based async patterns provide superior composability through Promise.all(), Promise.race(), and async/await syntax compared to callbacks
- Consistent concurrency model across all agents and utilities reduces cognitive load and integration complexity
- TypeScript's type system provides excellent support for Promise types, enabling better compile-time error detection and IDE support

## Consequences

Positive:
- Unified concurrency model across all external APIs improves developer experience and reduces integration errors
- Promise-based patterns enable better error handling with try-catch blocks and centralized error propagation
- Async/await syntax improves code readability and maintainability compared to callback chains
- Composable async operations through Promise combinators (all, race, allSettled) enable sophisticated orchestration patterns

Negative:
- Requires all developers to understand Promise semantics and async/await patterns thoroughly
- May introduce slight performance overhead compared to synchronous operations for trivial tasks
- Debugging async code can be more complex, requiring understanding of event loop and microtask queue behavior
- Legacy code using callbacks requires refactoring to align with this standard

## Alternatives

- Callback-based asynchronous patterns (Node.js error-first callbacks) (rejected)
  Rejected because: Callbacks lead to callback hell, poor error handling, and are incompatible with modern async/await syntax. The ecosystem has moved away from this pattern.
  When valid: Only when interfacing with legacy libraries that provide no Promise-based alternative
- Synchronous blocking APIs for all operations (rejected)
  Rejected because: Blocking operations would severely degrade performance in I/O-bound scenarios and prevent concurrent execution of independent tasks
  When valid: Only for pure computational functions with no I/O or side effects
- Observable/Stream-based patterns (RxJS) (rejected)
  Rejected because: Adds significant complexity and library dependency. Promises are sufficient for single-value async operations which dominate the codebase
  When valid: Could be reconsidered for event streams or multi-value async sequences if requirements change

## Risks

- Unhandled Promise rejections may cause silent failures or process crashes if not properly caught
  Mitigation: Implement global unhandled rejection handlers, enforce linting rules for floating promises, and require comprehensive error handling in all async methods
  Owner: Engineering team
- Inconsistent error types across different agents may complicate error handling at integration points
  Mitigation: Define standard error classes and establish error handling conventions documented in API guidelines
  Owner: API design team
- Performance degradation if async patterns are applied unnecessarily to synchronous operations
  Mitigation: Provide clear guidelines on when async is required vs optional, conduct performance reviews for critical paths
  Owner: Performance engineering team

## Implementation Notes

- Use TypeScript's Promise<T> type annotations for all async method return types to ensure type safety
- Prefer async/await over .then()/.catch() chains for better readability and error handling
- Always await or return promises in async functions to prevent floating promises
- Use Promise.all() for concurrent independent operations, Promise.allSettled() when all results are needed regardless of failures
- Configure ESLint rules: @typescript-eslint/no-floating-promises, @typescript-eslint/promise-function-async
- Document async behavior in JSDoc comments including @returns {Promise<Type>} and @throws annotations

## Continuation Context


Verify commands:
- grep -r 'async.*function\|Promise<' src/agents/ src/core/ src/lib.ts | wc -l
- npx eslint --rule '@typescript-eslint/no-floating-promises: error' src/
- grep -r 'callback.*function\|function.*callback' src/agents/ src/core/ && echo 'Found callback patterns' || echo 'No callback patterns found'

Accept when:
- All external API methods in agent implementations return Promise types or use async function declarations
- ESLint validation passes with no-floating-promises rule enabled across all source files
- No callback-based async patterns detected in public API surfaces (grep verification returns zero matches)
- All async methods include proper error handling with try-catch or .catch() handlers

## Enforcement

- Verified by: TypeScript compiler type checking enforces Promise return types
- Verified by: ESLint rules (@typescript-eslint/no-floating-promises, @typescript-eslint/promise-function-async) in CI pipeline
- Verified by: Code review checklist includes async pattern compliance verification
- Verified by: Automated grep-based pattern detection in pre-commit hooks
- Violation handling: CI build fails if ESLint async rules are violated
- Violation handling: Code review blocks merge if callback patterns detected in new external APIs
- Violation handling: Automated comments on PRs flag synchronous operations that should be async
- Violation handling: Quarterly codebase audits identify and track technical debt for legacy callback code
- Exception process: Developer submits exception request with performance benchmarks or technical justification
- Exception process: Architecture review board evaluates exception against policy EXC-001 or EXC-002 criteria
- Exception process: Approved exceptions documented in code with ADR reference and expiration date
- Exception process: Exception registry maintained in docs/adr/exceptions.md with periodic review