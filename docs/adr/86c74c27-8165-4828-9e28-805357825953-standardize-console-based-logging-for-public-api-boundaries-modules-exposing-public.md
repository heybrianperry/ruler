# Standardize Console-Based Logging for Public API Boundaries: Modules Exposing Public

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all modules that expose public API contracts or handle external integration points.

## Context

- The codebase exhibits consistent use of console-based logging (console.error, console.warn) across modules that define public API contracts and handle external integrations
- Five files demonstrate this pattern with 90.52% confidence, including core utilities (ConfigLoader, FileSystemUtils), CLI handlers, agent implementations (FirebenderAgent), and shared constants
- Logging statements are consistently prefixed with contextual markers ([ruler], [ruler:verbose], ERROR_PREFIX) to distinguish operational messages from application output
- The pattern emerges at boundaries where the system interacts with external configuration sources (XDG_CONFIG_HOME, process.env), file system operations, and user-facing CLI interfaces
- No structured logging framework or abstraction layer is present; direct console API usage provides immediate observability without additional dependencies

## Problem Statement

Public API boundaries and external integration points require consistent observability mechanisms to diagnose configuration loading failures, file system errors, and runtime warnings without introducing heavyweight logging dependencies or obscuring the separation between operational diagnostics and application output.

## Decision

1. MUST: All modules exposing public API contracts (identified by exported interfaces, types, or functions) MUST use console.error for error-level diagnostics and console.warn for warning-level diagnostics

## Policy Block

- MUST All modules exposing public API contracts (identified by exported interfaces, types, or functions) MUST use console.error for error-level diagnostics and console.warn for warning-level diagnostics

In scope:
- All TypeScript modules in src/ that export public interfaces, types, or functions
- CLI handler modules that interact with user input or command-line arguments
- Core utility modules that perform file system operations, configuration loading, or environment variable access
- Agent implementations that coordinate external resources or maintain state

Out of scope:
- Internal utility functions not exposed through public exports
- Test files and test utilities
- Build scripts and development tooling
- Third-party library code or vendored dependencies

Exceptions:
- EXC-001: A module requires structured logging for compliance, audit, or production monitoring requirements that cannot be satisfied by console-based logging

## Rationale

- The evidence shows consistent adoption of console-based logging across 5 files with 90.52% confidence, indicating an established pattern rather than isolated usage
- Direct console API usage eliminates external logging dependencies while providing immediate observability at critical boundaries (configuration loading, file system operations, CLI interactions)
- Prefixed logging statements (e.g., [ruler], ERROR_PREFIX) demonstrate intentional separation between operational diagnostics and application output, preserving clean stdout for programmatic consumption
- The pattern aligns with the module's role as a CLI tool and configuration processor where stderr-based diagnostics are conventional and expected by users

## Consequences

Positive:
- Zero external dependencies for observability, reducing bundle size and eliminating version conflicts with logging frameworks
- Immediate visibility into configuration loading failures, file system errors, and runtime warnings without requiring log aggregation infrastructure
- Clear separation between operational diagnostics (stderr) and application output (stdout) enables reliable programmatic consumption and shell scripting
- Low cognitive overhead for contributors familiar with standard console APIs, reducing onboarding friction

Negative:
- Lack of structured logging limits integration with centralized log aggregation systems or observability platforms
- No built-in log level filtering or runtime configuration without custom wrapper implementations
- Difficult to unit test logging behavior without mocking global console object
- Limited metadata capture (timestamps, stack traces, correlation IDs) compared to dedicated logging frameworks

## Alternatives

- Adopt a structured logging framework (e.g., winston, pino, bunyan) with configurable transports and log levels (rejected)
  Rejected because: Introduces external dependencies and complexity disproportionate to the current observability needs of a CLI tool; evidence shows console-based logging satisfies diagnostic requirements across all observed modules
  When valid: If the system evolves to require centralized log aggregation, correlation IDs, or production monitoring with structured metadata
- Implement a lightweight logging abstraction layer wrapping console APIs with configurable log levels (deferred)
  Rejected because: No evidence of log level filtering requirements in the current codebase; premature abstraction without demonstrated need
  When valid: If multiple modules require runtime-configurable log levels or if testing logging behavior becomes a maintenance burden
- Eliminate logging statements entirely and rely on error propagation through return values or exceptions (rejected)
  Rejected because: Removes critical observability at external boundaries where failures may occur outside application control (file system, environment variables, user input); evidence shows logging provides actionable diagnostics for non-fatal warnings
  When valid: Never appropriate for CLI tools and configuration processors where user-facing diagnostics are essential

## Risks

- Inconsistent logging patterns may emerge as the codebase grows, leading to fragmented observability and difficulty diagnosing issues across module boundaries
  Mitigation: Establish linting rules or code review guidelines to enforce prefixed console logging at public API boundaries; document logging conventions in contributor guidelines
  Owner: Engineering team
- Global console object mocking in tests may introduce brittle test dependencies and obscure actual logging behavior in integration scenarios
  Mitigation: Limit console mocking to unit tests that explicitly verify logging behavior; prefer integration tests that validate end-to-end functionality without mocking observability
  Owner: Engineering team
- Future requirements for structured logging or centralized observability may necessitate widespread refactoring if console-based logging becomes deeply embedded
  Mitigation: Monitor for signals that structured logging is needed (e.g., production deployment, multi-tenant usage, compliance requirements); plan migration path through abstraction layer if threshold is reached
  Owner: Architecture team

## Implementation Notes

- Define shared constants for logging prefixes (e.g., ERROR_PREFIX, WARN_PREFIX, VERBOSE_PREFIX) in src/constants.ts to ensure consistency across modules
- Use template literals to include contextual information (file paths, error messages, configuration keys) in logging statements for actionable diagnostics
- Reserve console.error for error-level diagnostics and console.warn for warning-level diagnostics; avoid console.log for operational messages to maintain clean stdout
- When implementing verbose logging modes, use conditional checks (e.g., if (verbose)) before console calls to avoid performance overhead in production

## Continuation Context


Verify commands:
- grep -r 'console\.error\|console\.warn' src/ --include='*.ts' | grep -E '\[(ruler|ERROR_PREFIX|WARN_PREFIX)' || echo 'No prefixed logging found'
- grep -r 'console\.log' src/ --include='*.ts' --exclude='*.test.ts' | grep -v 'test\|spec' && echo 'Found console.log in non-test files' || echo 'No console.log in production code'
- npm test -- --grep 'logging' 2>&1 | grep -E 'pass|fail' || echo 'No logging tests found'

Accept when:
- All console.error and console.warn statements in src/ modules with public exports include contextual prefixes identifying the subsystem
- No console.log statements exist in production code paths (excluding test files and development utilities)
- Logging statements at configuration loading and file system boundaries include actionable context (paths, error messages, keys)

## Enforcement

- Verified by: Code review checklist requiring verification of logging patterns in modules touching public API boundaries
- Verified by: Grep-based verification commands in CI pipeline to detect unprefixed console usage or console.log in production code
- Verified by: Manual inspection during architecture reviews for new modules exposing public contracts
- Violation handling: CI pipeline warnings for unprefixed console.error or console.warn usage in public API modules
- Violation handling: Code review feedback requesting addition of contextual prefixes or migration from console.log to appropriate error/warn levels
- Violation handling: Documentation updates if violations indicate the pattern is insufficient for emerging observability requirements
- Exception process: Document the specific observability requirement that cannot be satisfied by console-based logging in the module's header comments
- Exception process: Propose alternative logging approach (structured framework, custom abstraction) with rationale in architecture review
- Exception process: Update this ADR with exception details and conditions under which the alternative is valid