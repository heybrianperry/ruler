# Native Console API for Error and Warning Logging: Recoverable Errors Non Fatal Warnings Use

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires error and warning logging across multiple architectural layers including agents, CLI handlers, and core filesystem utilities
- Error conditions include file read/parse failures, CLI user errors, and filesystem operation failures that need to be surfaced to developers or users
- The project operates as a CLI tool or development-focused application where console output is the primary user interface
- No structured logging library dependency is present in the codebase, indicating a preference for zero-dependency or minimal-dependency approach for logging

## Problem Statement

The system needs a consistent mechanism to report error conditions and warnings across all architectural layers without introducing external logging library dependencies, while maintaining simplicity for CLI-based applications where console output is the primary feedback channel.

## Decision

1. MUST: Recoverable errors and non-fatal warnings MUST use console.warn() to distinguish them from fatal errors

## Policy Block

- MUST Recoverable errors and non-fatal warnings MUST use console.warn() to distinguish them from fatal errors

In scope:
- All error handling code across agent layer, CLI layer, and core utilities
- File I/O operations that may fail (read, parse, write)
- User-facing error messages in CLI handlers
- System-level errors in filesystem utilities

Out of scope:
- Success case logging or informational messages (not covered by current evidence)
- Debug-level logging (not observed in evidence)
- Structured log aggregation or remote logging systems
- Production server applications requiring log levels, rotation, or persistence

## Rationale

- The native Console API provides sufficient functionality for CLI tools and development-focused applications without external dependencies
- Zero-dependency logging reduces package size, eliminates supply chain risk, and simplifies the build process
- Console output is the natural feedback mechanism for CLI applications where users interact via terminal
- The warn/error distinction provides adequate severity levels for the observed error handling patterns across the codebase

## Consequences

Positive:
- Zero external dependencies for logging reduces package size and eliminates potential security vulnerabilities from logging libraries
- Native Console API is universally available in all JavaScript runtimes without configuration or setup
- Simple mental model for developers: warn for recoverable errors, error for fatal conditions
- Console output integrates naturally with CLI workflows and terminal-based debugging

Negative:
- No structured logging capabilities (log levels, JSON formatting, metadata) limit observability in production environments
- Cannot easily redirect logs to files, remote aggregators, or monitoring systems without custom wrappers
- No built-in log filtering, sampling, or rate limiting for high-volume scenarios
- Difficult to test logging behavior without mocking global console object

## Alternatives

- Adopt a structured logging library with log levels, formatters, and transport mechanisms (rejected)
  Rejected because: Evidence shows deliberate use of native Console API across all layers; no structured logging library is present, indicating preference for zero-dependency approach
  When valid: When the application evolves into a long-running service requiring log aggregation, filtering, or remote transport
- Create a custom logging abstraction layer wrapping console methods (rejected)
  Rejected because: Current implementation uses console methods directly without abstraction; adding a wrapper would increase complexity without evidence of need
  When valid: When multiple logging backends need to be supported or when testing requires log capture without global mocking
- Use process.stderr.write() and process.stdout.write() directly for more control (rejected)
  Rejected because: Console API provides automatic newline handling and error object formatting that would need manual implementation with direct stream writes
  When valid: When precise control over output buffering or formatting is required for performance-critical logging

## Risks

- Console output may be suppressed or redirected in production environments, causing silent failures if errors are not handled
  Mitigation: Ensure all error paths have appropriate error handling beyond logging; use exit codes for CLI failures
  Owner: engineering team
- Lack of structured logging makes it difficult to parse logs programmatically or integrate with monitoring systems
  Mitigation: Document this limitation; plan migration path to structured logging if application scope expands beyond CLI tool
  Owner: engineering team
- Testing code that logs to console requires global mocking, which can be fragile and affect other tests
  Mitigation: Isolate logging tests; consider test utilities that capture console output; document testing patterns for logged code
  Owner: engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When catching exceptions, pass the error object as a second argument to console methods to preserve stack traces for debugging
- Use template literals for message formatting to include contextual information such as file paths, operation names, or component identifiers
- Consider prefixing messages with component identifiers in square brackets to aid log filtering in complex workflows

## Continuation Context


Verify commands:
- Discover and inspect source files in agent, CLI, and core utility directories to confirm all error logging uses console.warn or console.error
- Search the codebase for console method calls and verify no external logging library imports are present
- Review error handling paths to confirm warn/error distinction aligns with severity (recoverable vs fatal)

Accept when:
- All error and warning logging across agent, CLI, and core layers uses console.warn or console.error exclusively
- No external logging library dependencies are present in dependency manifests
- Error severity distinction is consistent: console.warn for recoverable errors, console.error for fatal errors

## Enforcement

- Verified by: Code review verification that new error handling uses console.warn or console.error
- Verified by: Static analysis or linting rules to detect external logging library imports
- Verified by: Manual inspection of error handling patterns during pull request review
- Violation handling: Pull requests introducing external logging libraries must provide architectural justification
- Violation handling: Code using alternative logging mechanisms should be refactored to use Console API during review
- Violation handling: Violations discovered post-merge should be documented as technical debt and prioritized for remediation
- Exception process: Exceptions may be granted for specific modules requiring structured logging or remote transport
- Exception process: Exception requests must document why Console API is insufficient for the use case
- Exception process: Approved exceptions should be isolated to specific modules and documented in code comments