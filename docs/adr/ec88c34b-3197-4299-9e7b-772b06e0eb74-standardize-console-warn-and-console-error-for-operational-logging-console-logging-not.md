# Standardize console.warn and console.error for Operational Logging: Console Logging Not

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all agent implementations, CLI handlers, configuration loaders, and file system utilities within the ruler codebase.

## Context

- The codebase operates in a CLI and agent-based environment where runtime diagnostics must be immediately visible to operators without external logging infrastructure
- Configuration loading, file system operations, and agent initialization involve recoverable errors and warnings that require user notification but should not halt execution
- The system processes user-provided configuration files (TOML, JSON) and file system paths where parse failures and missing files are expected operational conditions
- Four core modules (FirebenderAgent, ConfigLoader, handlers, FileSystemUtils) independently adopted console.warn and console.error for operational feedback, establishing an emergent pattern

## Problem Statement

Without a standardized logging approach, operational diagnostics across agents, configuration loaders, and file system utilities risk inconsistent error reporting, making it difficult for users to distinguish between recoverable warnings and critical failures during CLI operations and agent execution.

## Decision

1. MUST_NOT: Console logging MUST NOT be used for structured telemetry, metrics, or audit trails that require persistence or aggregation

## Policy Block

- MUST_NOT Console logging MUST NOT be used for structured telemetry, metrics, or audit trails that require persistence or aggregation

In scope:
- Agent implementations (FirebenderAgent and similar)
- Configuration loaders (ConfigLoader, TOML/JSON parsing)
- CLI command handlers (applyHandler, initHandler, revertHandler)
- File system utilities (findRulerDir, readMarkdownFiles, writeGeneratedFile)
- Initialization and bootstrap code paths
- User-facing error conditions during runtime operations

Out of scope:
- Structured application telemetry or observability pipelines
- Audit logs requiring persistence or compliance tracking
- Performance metrics or profiling data
- Debug logging in non-production environments (use dedicated debug tooling)
- Third-party library internal logging

## Rationale

- The pattern appears consistently across 4 files with 90.03% confidence, indicating organic adoption as the de facto operational logging mechanism
- Console-based logging provides immediate feedback in CLI environments without requiring external logging infrastructure, reducing operational complexity for a developer-facing tool
- The distinction between console.warn (recoverable) and console.error (critical) aligns with the operational semantics of configuration loading and file system operations where graceful degradation is expected
- Standardizing on native console methods avoids introducing logging library dependencies while maintaining sufficient diagnostic capability for the tool's operational scope

## Consequences

Positive:
- Immediate visibility of operational issues to CLI users without requiring log file inspection or external tooling
- Zero external dependencies for logging infrastructure, reducing bundle size and deployment complexity
- Clear semantic distinction between warnings (console.warn) and errors (console.error) aids user triage and response
- Consistent error reporting across agents, configuration loaders, and file system utilities improves user experience and troubleshooting efficiency

Negative:
- Console output is ephemeral and cannot be aggregated, searched, or analyzed for operational trends without external capture mechanisms
- No structured logging format limits integration with observability platforms or automated error tracking systems
- Console logging in production environments may expose sensitive file paths or configuration details in user terminals
- Lack of log levels beyond warn/error reduces granularity for debugging complex operational scenarios

## Alternatives

- Adopt a structured logging library (e.g., winston, pino) with configurable transports and log levels (rejected)
  Rejected because: Introduces external dependencies and configuration complexity disproportionate to the operational needs of a CLI tool where immediate console feedback is sufficient
  When valid: Valid if the system evolves to require persistent logs, structured telemetry, or integration with centralized observability platforms
- Implement a custom logging abstraction layer wrapping console methods with additional metadata and formatting (rejected)
  Rejected because: Adds maintenance burden and code complexity without addressing the core requirement for immediate user-facing diagnostics in a CLI context
  When valid: Valid if multiple output targets (console, file, remote) or advanced formatting (colors, timestamps, structured fields) become requirements
- Use process.stderr.write and process.stdout.write for direct stream control (rejected)
  Rejected because: Loses the semantic distinction between warnings and errors provided by console.warn/console.error and requires manual formatting
  When valid: Valid if precise control over output streams or buffering behavior becomes necessary for performance or integration reasons

## Risks

- Sensitive file paths or configuration values may be exposed in console output visible to users or captured in terminal logs
  Mitigation: Implement path sanitization and redaction for sensitive values before logging; document security considerations for console output in shared environments
  Owner: engineering team
- Lack of structured logging limits future integration with error tracking services (e.g., Sentry) or observability platforms
  Mitigation: Design console logging calls to be easily replaceable with structured logging adapters; maintain clear separation between operational logging and business logic
  Owner: engineering team
- Inconsistent message formatting across modules reduces user comprehension and makes automated parsing difficult
  Mitigation: Establish and document message format conventions (prefixes, error context, file paths); enforce through code review and linting rules
  Owner: engineering team

## Implementation Notes

- Define constants for common prefixes (e.g., ERROR_PREFIX, WARN_PREFIX) in a shared constants module to ensure consistency across CLI handlers
- Include contextual information in all console messages: operation name, file path, and original error message to aid user troubleshooting
- Use console.warn for recoverable conditions (missing optional config, fallback behavior) and console.error for conditions requiring user intervention
- Consider wrapping console calls in utility functions (e.g., logWarning, logError) to centralize formatting logic and enable future migration to structured logging

## Continuation Context


Verify commands:
- grep -r 'console\.warn' src/ | grep -E '(ConfigLoader|FirebenderAgent|handlers|FileSystemUtils)' | wc -l
- grep -r 'console\.error' src/ | grep -E '(ConfigLoader|FirebenderAgent|handlers|FileSystemUtils)' | wc -l
- grep -r 'console\.log' src/ | grep -v test | wc -l

Accept when:
- All configuration loading failures and file system errors in core modules use console.warn or console.error appropriately
- CLI handlers consistently use ERROR_PREFIX or similar constants for error messages
- No console.log usage exists in production code paths (only console.warn and console.error for operational diagnostics)

## Enforcement

- Verified by: Code review checklist requiring console.warn/console.error usage for operational diagnostics
- Verified by: ESLint rule restricting console.log in production code (allow only console.warn and console.error)
- Verified by: Grep-based verification in CI pipeline checking for console method usage patterns
- Violation handling: CI pipeline fails if console.log is detected in non-test production code
- Violation handling: Code review requires justification for any console usage outside established patterns
- Violation handling: Violations in merged code trigger refactoring tickets to align with standardized logging approach
- Exception process: Exceptions for specialized diagnostic output must be documented in code comments with rationale
- Exception process: Temporary debug logging during incident response may bypass rules but must be removed before merge
- Exception process: Proposals for alternative logging approaches require ADR amendment with evidence of operational need