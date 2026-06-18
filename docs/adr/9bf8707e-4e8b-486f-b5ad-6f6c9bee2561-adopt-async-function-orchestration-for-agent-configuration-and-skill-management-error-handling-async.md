# Adopt Async Function Orchestration for Agent Configuration and Skill Management: Error Handling Async

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent configuration and skill processing operations.

## Context

- Agent configuration management (FirebenderAgent) requires coordinated file I/O operations including reading, parsing, and writing JSON configuration files with error handling for missing or malformed data
- Skill discovery and propagation (SkillsProcessor) involves multiple asynchronous operations across the filesystem including directory traversal, file discovery, cleanup, and propagation to multiple targets (Claude, Codex)
- Both components use Node.js fs/fs.promises APIs for file system operations and require coordination of multiple async operations through named functions
- The codebase separates concerns by organizing async workflows into discrete, named functions (applyRulerConfig, loadExistingConfig, saveConfig, discoverSkills, propagateSkills) rather than inline promises or callbacks

## Problem Statement

Without a consistent concurrency model for coordinating multiple asynchronous file system operations, agent configuration and skill management workflows risk callback hell, error handling inconsistencies, and difficult-to-maintain code. The system needs a standardized approach to orchestrate async operations that maintains readability, enables proper error propagation, and supports testing.

## Decision

1. SHOULD: Error handling for async file operations SHOULD use try-catch blocks with fallback behavior (e.g., console.warn for non-critical failures, default values for missing configs)

## Policy Block

- SHOULD Error handling for async file operations SHOULD use try-catch blocks with fallback behavior (e.g., console.warn for non-critical failures, default values for missing configs)

In scope:
- FirebenderAgent configuration loading, parsing, and persistence
- SkillsProcessor discovery, cleanup, and propagation workflows
- Any agent or core module performing coordinated file system operations
- MCP (Model Context Protocol) configuration handling

Out of scope:
- Synchronous utility functions that do not perform I/O
- Third-party library internal implementations
- Simple single-operation file reads that do not require coordination

Exceptions:
- EXC-001: Legacy code modules scheduled for deprecation within one release cycle

## Rationale

- The evidence shows consistent use of named async functions (applyRulerConfig, loadExistingConfig, saveConfig, discoverSkills, propagateSkills) across both FirebenderAgent and SkillsProcessor, indicating an established pattern for managing async complexity
- Both modules use fs.promises and path APIs for file operations, demonstrating commitment to promise-based async patterns over callbacks
- The pattern of using Set.add for deduplication (seen.add(key), targets.add(target)) within async workflows indicates awareness of state management concerns in concurrent operations
- Error handling with console.warn and JSON.parse try-catch patterns shows defensive programming for non-critical failures while maintaining workflow continuity

## Consequences

Positive:
- Improved code readability and maintainability through named, testable async functions
- Consistent error handling patterns across agent configuration and skill management
- Reduced risk of callback hell and promise chain complexity
- Better testability through isolated, named async functions

Negative:
- Requires developers to understand async/await semantics and promise behavior
- May introduce performance overhead if sequential operations could be parallelized
- Additional complexity in error propagation across multiple async function boundaries
- Potential for unhandled promise rejections if error handling is incomplete

## Alternatives

- Use callback-based fs APIs with nested callbacks (rejected)
  Rejected because: Leads to callback hell, poor error handling, and reduced readability; promise-based APIs are the modern Node.js standard
  When valid: Never valid for new code; only acceptable in legacy modules scheduled for deprecation
- Use inline promise chains with .then()/.catch() instead of async/await (rejected)
  Rejected because: Less readable than async/await, harder to debug, and more error-prone for complex workflows with multiple sequential operations
  When valid: May be acceptable for simple single-operation promises where async/await adds no value
- Use reactive streams (RxJS) for all async operations (rejected)
  Rejected because: Adds significant complexity and dependency weight for workflows that are primarily sequential file operations, not event streams
  When valid: Could be reconsidered if requirements shift to real-time event processing or complex backpressure scenarios

## Risks

- Unhandled promise rejections in async functions could crash the Node.js process or leave operations in inconsistent state
  Mitigation: Implement comprehensive try-catch blocks in all async functions; enable Node.js unhandledRejection warnings; add integration tests for error paths
  Owner: Engineering team
- Sequential async operations may introduce performance bottlenecks when operations could be parallelized
  Mitigation: Profile critical paths; use Promise.all for independent operations; document intentional sequencing for state safety
  Owner: Engineering team
- Inconsistent error handling across modules could lead to silent failures or poor user experience
  Mitigation: Establish error handling guidelines; use linting rules to enforce try-catch; implement centralized error logging
  Owner: Engineering team

## Implementation Notes

- Use fs.promises for all file system operations; import as 'import { promises as fs } from "fs"' or 'import fs from "fs/promises"'
- Name async functions descriptively based on their workflow purpose (e.g., loadExistingConfig, propagateSkillsForClaude) to improve code navigation and testing
- Wrap JSON.parse calls in try-catch blocks and provide sensible defaults or log warnings for parse failures
- Use Set or Map for deduplication in async workflows to prevent duplicate processing; ensure state mutations complete before dependent operations
- Consider Promise.all for independent async operations (e.g., propagating to multiple targets) to improve performance

## Continuation Context


Verify commands:
- grep -r "fs\.promises\|from 'fs/promises'" src/agents src/core --include='*.ts' | wc -l
- grep -r "async function\|async .*=>" src/agents/FirebenderAgent.ts src/core/SkillsProcessor.ts | wc -l
- grep -r "JSON\.parse" src/agents src/core --include='*.ts' -A 2 | grep -c "try\|catch"

Accept when:
- All file I/O operations in FirebenderAgent and SkillsProcessor use fs.promises or equivalent promise-based APIs
- Complex async workflows are decomposed into at least 3 named async functions per module
- At least 80% of JSON.parse operations are wrapped in try-catch blocks with error handling

## Enforcement

- Verified by: Code review checklist requiring async/await pattern verification
- Verified by: ESLint rules enforcing promise-based fs APIs and async function naming conventions
- Verified by: Integration tests covering async error paths and workflow coordination
- Violation handling: CI pipeline fails if callback-based fs APIs are detected in new code
- Violation handling: Code review blocks merge if async functions lack proper error handling
- Violation handling: Automated comments on PRs identifying promise anti-patterns
- Exception process: Developer documents exception rationale in PR description with tech lead tag
- Exception process: Tech lead reviews and approves with documented migration plan if legacy code
- Exception process: Exception logged in ADR exceptions registry with target resolution date