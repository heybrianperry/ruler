# Adopt Async/Await Concurrency Model for External API Integration: Parallel Independent Operations

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all agent implementations and external API integrations within the codebase. All new agents and API integration code MUST follow the async/await concurrency model as specified herein.

## Context

- The codebase contains multiple agent implementations (AiderAgent, AgentsMdAgent, AugmentCodeAgent, CopilotAgent, CursorAgent) that interact with external APIs and perform I/O-bound operations
- Pattern signature 9fc89443136b4a952c8158001bf1005f detected across 12 files with 90% confidence, indicating a consistent concurrency approach
- TypeScript/Node.js environment requires non-blocking I/O patterns to maintain responsiveness when integrating with external services
- Core utilities (SkillsUtils, SkillsProcessor, GitignoreUtils, revert-engine) demonstrate consistent use of async/await paradigm for file system operations and external process execution
- Test harness and library entry points (lib.ts, revert.ts, tests/harness.ts) establish async/await as the foundational concurrency model for the entire system

## Problem Statement

When integrating with external APIs and performing I/O-bound operations across multiple agent implementations, the system needs a consistent, maintainable, and performant concurrency model that prevents callback hell, enables proper error handling, and maintains code readability while ensuring non-blocking execution in a Node.js environment.

## Decision

1. SHOULD: Parallel independent operations SHOULD use Promise.all() or Promise.allSettled() with await to maximize concurrency

## Policy Block

- SHOULD Parallel independent operations SHOULD use Promise.all() or Promise.allSettled() with await to maximize concurrency

In scope:
- All agent implementations (AiderAgent, AgentsMdAgent, AugmentCodeAgent, CopilotAgent, CursorAgent)
- Core utility modules (SkillsUtils, SkillsProcessor, GitignoreUtils, revert-engine)
- Library entry points and public APIs (lib.ts, revert.ts)
- Test harnesses and integration tests (tests/harness.ts)
- Any new code that performs I/O operations, network requests, or external API calls

Out of scope:
- Pure synchronous utility functions that perform only CPU-bound operations
- Event emitter callbacks that must remain synchronous by design
- Third-party library code that uses different concurrency patterns (must be wrapped/adapted)
- Legacy callback-based code scheduled for refactoring (must be documented with migration plan)

Exceptions:
- EXC-001: Integrating with third-party libraries that only provide callback-based APIs
- EXC-002: Performance-critical hot paths where async overhead is measurably significant

## Rationale

- Pattern detected across 12 files with 90% confidence indicates this is an established architectural standard in the codebase
- Async/await provides superior readability and maintainability compared to callback chains or raw Promise .then() chains, especially for complex agent workflows
- TypeScript's type system provides excellent support for async/await with proper Promise type inference and error handling
- The facet 'paradigm.concurrency_model' explicitly identifies this as a fundamental architectural decision affecting how the system handles concurrent operations

## Consequences

Positive:
- Improved code readability with linear, synchronous-looking code flow that is easier to understand and maintain
- Consistent error handling patterns across all agent implementations using standard try-catch blocks
- Better debugging experience with clearer stack traces compared to callback-based code
- Enables proper resource cleanup with try-finally blocks and reduces risk of resource leaks
- Facilitates parallel execution of independent operations using Promise.all() while maintaining readable code

Negative:
- Requires all developers to understand Promise semantics and async/await behavior, including event loop implications
- Potential for performance overhead in extremely high-frequency operations compared to synchronous alternatives
- Risk of unhandled promise rejections if error handling is not properly implemented at all await sites
- May complicate integration with older callback-based libraries requiring promisification wrappers

## Alternatives

- Use raw Promises with .then()/.catch() chains for all asynchronous operations (rejected)
  Rejected because: Promise chains lead to less readable code, especially with complex control flow, and are more error-prone. The detected pattern shows the team has already moved away from this approach.
  When valid: Only when integrating with legacy code that cannot be easily refactored
- Use callback-based patterns with error-first callbacks (Node.js style) (rejected)
  Rejected because: Callback-based patterns lead to callback hell, make error handling complex, and are difficult to compose. Modern TypeScript/Node.js strongly favors Promises and async/await.
  When valid: Never for new code; only acceptable when wrapping third-party callback-based APIs
- Use reactive programming with RxJS Observables for all async operations (rejected)
  Rejected because: RxJS adds significant complexity and learning curve for operations that don't require streaming or complex event composition. The detected pattern shows async/await is sufficient for current needs.
  When valid: Could be considered for future streaming API integrations or complex event-driven workflows

## Risks

- Unhandled promise rejections causing process crashes or silent failures in production
  Mitigation: Implement global unhandled rejection handlers, enforce try-catch in code reviews, and add linting rules to detect missing error handling
  Owner: Engineering team
- Performance degradation from excessive sequential awaits where parallel execution is possible
  Mitigation: Establish code review guidelines to identify parallelization opportunities, provide training on Promise.all() patterns, and include performance testing in CI
  Owner: Engineering team
- Inconsistent error handling patterns across different agent implementations leading to maintenance burden
  Mitigation: Create shared error handling utilities, document standard patterns in developer guidelines, and enforce consistency through code review checklist
  Owner: Engineering team

## Implementation Notes

- Use TypeScript's strict mode to ensure all async functions have proper return type annotations (Promise<T>)
- Establish a pattern library with common async/await idioms for the team (parallel execution, error handling, timeout patterns)
- Consider implementing utility functions for common patterns like retry logic, timeout wrappers, and circuit breakers
- Document the async/await pattern in onboarding materials and provide examples from existing agent implementations
- Use ESLint rules like 'no-floating-promises' and 'require-await' to catch common async/await mistakes

## Continuation Context


Verify commands:
- grep -r 'async.*function\|async.*(' src/ --include='*.ts' | wc -l
- grep -r '\.then(' src/ --include='*.ts' | grep -v 'node_modules' | wc -l
- npx eslint src/ --rule 'no-floating-promises: error' --rule 'require-await: warn'

Accept when:
- All agent implementations (AiderAgent, AgentsMdAgent, AugmentCodeAgent, CopilotAgent, CursorAgent) use async/await for external API calls
- Core utility modules expose async interfaces and use await for I/O operations
- ESLint checks pass with no-floating-promises and require-await rules enabled
- Code review checklist includes verification of proper async/await error handling

## Enforcement

- Verified by: Automated ESLint checks in CI pipeline with no-floating-promises and require-await rules
- Verified by: Code review checklist requiring verification of async/await usage and error handling
- Verified by: TypeScript compiler strict mode ensuring proper Promise type annotations
- Violation handling: CI build fails if ESLint rules detect floating promises or missing error handling
- Violation handling: Code review blocks merge if async/await patterns are not followed in new code
- Violation handling: Legacy code violations are documented in technical debt backlog with migration plan
- Exception process: Developer submits exception request with justification to tech lead
- Exception process: Tech lead reviews with consideration of performance data or third-party integration constraints
- Exception process: Approved exceptions must be documented in code with ADR reference and rationale
- Exception process: Exception review occurs quarterly to assess if circumstances have changed