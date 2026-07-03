# Standardize Core Node.js Module Imports for Test Infrastructure and Agent Implementation: Test Files Import

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all test suites, agent implementations, and core infrastructure modules within the Ruler project.

## Context

- The codebase demonstrates consistent usage of Node.js core modules across 67 files with 91.44% confidence, indicating an established architectural pattern rather than ad-hoc choices.
- Test infrastructure files (tests/*.test.ts) systematically import 'fs/promises', 'path', 'os', and 'child_process' for test harness setup, project scaffolding, and CLI invocation.
- Agent implementation files (src/agents/*.ts) consistently import 'path', 'fs', and define standard interfaces (IAgent, IAgentConfig) for configuration management and file system operations.
- The pattern spans both test and production code, suggesting a deliberate architectural decision to rely on Node.js built-in modules for core file system, path manipulation, and process management operations.
- Evidence shows integration with testing frameworks (describe, it, beforeEach, afterEach) alongside core module usage, indicating these modules form the foundation for both runtime and test-time operations.

## Problem Statement

Without standardized core module imports, the codebase risks inconsistent file system operations, path handling, and process management across different agents and test suites. This inconsistency can lead to platform-specific bugs, difficult-to-maintain test infrastructure, and unpredictable behavior when agents interact with configuration files and external processes.

## Decision

1. MUST: All test files MUST import 'fs/promises' for asynchronous file system operations rather than callback-based 'fs' APIs.

## Policy Block

- MUST All test files MUST import 'fs/promises' for asynchronous file system operations rather than callback-based 'fs' APIs.

In scope:
- All test files under tests/ directory including unit, integration, and CLI tests
- All agent implementation files under src/agents/ directory
- Core infrastructure modules in src/core/ that perform file system operations
- Utility modules (harness.ts, FileSystemUtils.ts) that provide shared functionality
- Configuration loading and revert operations in src/revert.ts and src/core/ConfigLoader.ts

Out of scope:
- Third-party library integrations that provide their own file system abstractions
- Browser-compatible modules that cannot use Node.js core APIs
- External tool wrappers that delegate file operations to external processes
- Mock implementations in test fixtures that simulate file system behavior

Exceptions:
- EXC-001: Agent implementations require synchronous file system operations during initialization or constructor execution where async/await is not available
- EXC-002: Legacy agent implementations (e.g., GeminiCliAgent) use synchronous fs operations for backward compatibility with existing configuration formats

## Rationale

- The 91.44% confidence across 67 files demonstrates this is an established architectural pattern, not an emerging trend, indicating deliberate design choices that should be codified.
- Consistent use of 'fs/promises' in test infrastructure ensures all file operations are async-first, preventing blocking operations during test execution and enabling parallel test runs.
- Standardizing on Node.js core modules eliminates external dependencies for fundamental operations, reducing supply chain risk and ensuring compatibility across Node.js versions.
- The pattern of importing 'path' universally prevents platform-specific path handling bugs (Windows vs. Unix) that commonly plague cross-platform Node.js applications.

## Consequences

Positive:
- Consistent file system operations across all agents and tests reduce platform-specific bugs and improve cross-platform reliability.
- Async-first approach with 'fs/promises' enables non-blocking I/O, improving performance in test suites and agent configuration operations.
- Zero external dependencies for core operations reduces attack surface, simplifies dependency management, and improves long-term maintainability.
- Standardized import patterns make code reviews faster and reduce cognitive load when switching between different modules.

Negative:
- Strict adherence to async patterns may complicate simple synchronous use cases, requiring additional boilerplate for error handling and promise resolution.
- Node.js core modules lack some convenience features provided by third-party libraries (e.g., fs-extra), potentially requiring more verbose code for complex operations.
- Legacy agent implementations may require refactoring to align with async-first patterns, creating technical debt and migration effort.
- Testing framework integration requires careful handling of async operations in beforeEach/afterEach hooks, increasing test complexity.

## Alternatives

- Use fs-extra library for enhanced file system operations with additional utilities like ensureDir, copy, and move (rejected)
  Rejected because: Introduces external dependency for operations that can be implemented with core modules; increases supply chain risk and bundle size without sufficient benefit given current usage patterns
  When valid: Valid if the codebase requires extensive directory tree operations, atomic moves, or recursive copy operations that would be verbose to implement with core modules
- Allow mixed usage of synchronous and asynchronous file system APIs based on developer preference (rejected)
  Rejected because: Creates inconsistent patterns across the codebase, makes performance characteristics unpredictable, and complicates testing infrastructure that expects async operations
  When valid: Valid only for initialization code paths where async context is unavailable and operations are guaranteed to be fast (e.g., reading small config files during module load)
- Create internal abstraction layer wrapping Node.js core modules with project-specific utilities (deferred)
  Rejected because: Not rejected but deferred; current evidence shows direct core module usage is sufficient, but abstraction may be valuable if cross-platform issues emerge or testing requires more sophisticated mocking
  When valid: Valid if the project needs to support multiple runtime environments (Node.js, Deno, Bun) or requires extensive mocking capabilities beyond what core modules provide

## Risks

- Node.js core module APIs may change between major versions, requiring updates across 67+ files when upgrading Node.js runtime
  Mitigation: Pin Node.js version in package.json engines field; establish automated testing across supported Node.js versions in CI; document minimum required Node.js version in README
  Owner: Engineering team
- Async-first approach may introduce race conditions in test setup/teardown if not properly sequenced with await
  Mitigation: Enforce ESLint rules for floating promises; require explicit await in all test lifecycle hooks; implement test harness utilities that enforce proper async sequencing
  Owner: Test infrastructure team
- Legacy agent implementations using synchronous fs operations may block event loop during configuration operations, degrading performance
  Mitigation: Audit all agent implementations for synchronous fs usage; create migration guide for converting to async patterns; add performance monitoring to detect blocking operations
  Owner: Agent development team

## Implementation Notes

- Use 'import fs from "fs/promises"' or 'import { readFile, writeFile } from "fs/promises"' for all new test files and agent implementations.
- When creating temporary directories in tests, use 'await fs.mkdtemp(path.join(os.tmpdir(), "ruler-"))' pattern as demonstrated in tests/integration/skills-mcp.test.ts.
- For CLI integration tests, use 'child_process' with inherited stdio or captured output as shown in tests/cli-no-mcp.test.ts for consistent command execution.
- Agent implementations should follow the pattern in src/agents/AbstractAgent.ts for importing 'path' and defining standard interfaces from './IAgent' or '../types'.

## Continuation Context


Verify commands:
- grep -r "from 'fs'" src/agents/ tests/ | grep -v "fs/promises" | grep -v "fs-extra" || echo 'No synchronous fs imports found'
- grep -r "require('path')\|from 'path'" src/agents/ tests/ | wc -l
- grep -r "fs\.readFileSync\|fs\.writeFileSync" src/agents/ || echo 'No synchronous fs operations in agents'
- npm test -- --testNamePattern="(claude-mcp-config|cli-no-mcp|skills-config-precedence)" --passWithNoTests

Accept when:
- All agent implementations under src/agents/ import 'path' and use 'fs/promises' for file operations, with zero synchronous fs calls in production code paths
- All test files under tests/ import 'fs/promises', 'path', and 'os' as needed, with consistent async patterns in beforeEach/afterEach hooks
- Grep verification commands confirm no synchronous fs operations in agent code and consistent core module import patterns across the codebase

## Enforcement

- Verified by: ESLint rules enforcing 'fs/promises' imports and flagging synchronous fs operations in production code
- Verified by: Code review checklist requiring verification of async patterns in new agent implementations
- Verified by: CI pipeline running grep-based verification commands to detect policy violations before merge
- Verified by: Automated dependency analysis ensuring no third-party file system libraries are introduced without architectural review
- Violation handling: CI build fails if synchronous fs operations are detected in src/agents/ directory
- Violation handling: Pull requests introducing non-standard core module usage patterns are flagged for architectural review
- Violation handling: Existing violations in legacy code are tracked as technical debt items with migration plans
- Violation handling: Performance regression tests alert if blocking operations are introduced in agent configuration paths
- Exception process: Developer submits exception request documenting why async pattern cannot be used and performance implications
- Exception process: Architecture review board evaluates exception against policy_exceptions criteria (EXC-001, EXC-002)
- Exception process: Approved exceptions are documented in code comments with ADR reference and expiration date or migration timeline
- Exception process: Exception usage is tracked in architectural decision log and reviewed quarterly for deprecation opportunities