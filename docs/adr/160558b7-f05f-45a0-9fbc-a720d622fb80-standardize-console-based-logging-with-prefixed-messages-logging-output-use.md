# Standardize Console-Based Logging with Prefixed Messages: Logging Output Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses console-based logging (console.error, console.warn) as the primary observability mechanism across multiple modules including constants.ts, FileSystemUtils.ts, and CLI handlers
- Logging messages are consistently prefixed with identifiers like '[ruler:verbose]', '[ruler]', and ERROR_PREFIX to distinguish log sources and severity levels
- Public API contracts export logging utilities (logVerbose, logInfo, logWarn) and error creation functions (createRulerError) to standardize logging behavior across the application
- The logging pattern appears in core infrastructure (file system operations), CLI handlers (user-facing errors), and utility modules, indicating system-wide adoption
- No structured logging framework or external logging library is detected in the evidence; the pattern relies on native console APIs

## Problem Statement

The application requires a consistent, lightweight logging mechanism that provides clear message attribution, severity differentiation, and debugging support without introducing external dependencies or complex logging infrastructure.

## Decision

1. MUST: All logging output MUST use native console methods (console.error, console.warn, console.log) without external logging frameworks

## Policy Block

- MUST All logging output MUST use native console methods (console.error, console.warn, console.log) without external logging frameworks

In scope:
- All TypeScript modules in src/ directory
- CLI handlers and user-facing error messages
- Core infrastructure modules (FileSystemUtils, constants)
- Error handling and diagnostic logging

Out of scope:
- Third-party library logging (libraries may use their own logging mechanisms)
- Browser-based logging if the application runs in browser contexts
- Production telemetry or metrics collection (this ADR covers diagnostic logging only)

## Rationale

- The evidence shows consistent adoption of console-based logging with prefixed messages across 3 files with 90.33% confidence, indicating an established pattern
- Native console APIs eliminate external dependencies, reduce bundle size, and provide immediate availability in Node.js environments
- Prefixed messages enable log filtering, source identification, and severity-based routing without structured logging overhead
- Exported logging utilities (logVerbose, logInfo, logWarn) in the public API contract demonstrate intentional standardization rather than ad-hoc logging

## Consequences

Positive:
- Zero external dependencies for logging reduces maintenance burden and security surface area
- Consistent message prefixes enable easy log filtering and parsing with standard Unix tools (grep, awk)
- Native console methods provide automatic stderr/stdout routing for error and warning levels
- Centralized logging utilities ensure uniform formatting and reduce code duplication

Negative:
- Console-based logging lacks structured logging features (JSON output, log levels, metadata fields) available in dedicated frameworks
- No built-in log rotation, persistence, or aggregation capabilities for production environments
- Limited observability compared to structured logging solutions that integrate with monitoring platforms
- String concatenation and template literals may have performance implications in high-throughput scenarios

## Alternatives

- Adopt a structured logging library (e.g., winston, pino, bunyan) (rejected)
  Rejected because: Evidence shows deliberate use of native console APIs across multiple modules; introducing a logging framework would add dependency weight and complexity not justified by current requirements
  When valid: Valid if production observability requirements demand structured logs, log levels, or integration with centralized logging platforms
- Use debug module for conditional logging (rejected)
  Rejected because: The pattern uses explicit logging functions (logVerbose, logInfo, logWarn) exported in public API contracts, indicating preference for application-controlled logging over environment-based debug flags
  When valid: Valid for development-only diagnostic logging that should be disabled in production
- Implement custom Logger class with configurable transports (rejected)
  Rejected because: Current evidence shows simple function-based logging utilities; a class-based abstraction would introduce unnecessary complexity for the observed use cases
  When valid: Valid if requirements evolve to support multiple output destinations, log buffering, or runtime log level configuration

## Risks

- Console-based logging may not scale to production observability needs if the application requires log aggregation, searching, or alerting
  Mitigation: Document logging strategy and establish criteria for when to migrate to structured logging; ensure log messages remain parseable with standard tools
  Owner: engineering team
- Inconsistent prefix usage across modules could reduce log filtering effectiveness and debugging efficiency
  Mitigation: Enforce use of centralized logging utilities (logVerbose, logInfo, logWarn) through code review and linting rules; document prefix conventions
  Owner: engineering team
- Native console methods provide no log level filtering, potentially exposing verbose logs in production
  Mitigation: Implement environment-based guards in logging utilities to suppress verbose logs in production; consider adding a LOG_LEVEL configuration option
  Owner: engineering team

## Implementation Notes

- Centralize all logging utilities in src/constants.ts or a dedicated logging module to ensure consistent formatting and prefix usage
- Define standard prefixes (e.g., '[ruler]', '[ruler:verbose]', ERROR_PREFIX) as exported constants to prevent string literal duplication
- Wrap console methods in utility functions (logVerbose, logInfo, logWarn, logError) to enable future enhancements like log level filtering or output redirection
- Document logging conventions in developer guidelines, including when to use each severity level and how to format multi-line error messages

## Continuation Context


Verify commands:
- grep -r 'console\.(log|error|warn)' src/ | grep -v 'logVerbose\|logInfo\|logWarn' | wc -l | awk '{if ($1 > 5) exit 1}'
- grep -r 'export.*log(Verbose|Info|Warn|Error)' src/constants.ts
- grep -r 'console\.error.*\[ruler' src/ | wc -l | awk '{if ($1 > 0) exit 0; else exit 1}'

Accept when:
- All console logging calls use centralized utility functions (logVerbose, logInfo, logWarn) exported from constants or logging module
- Log messages include consistent prefixes that identify source and severity
- Error-level messages use console.error and warning-level messages use console.warn

## Enforcement

- Verified by: Code review checking for direct console method usage outside utility functions
- Verified by: Grep-based verification commands in CI pipeline to detect non-compliant logging patterns
- Verified by: Manual inspection of log output during integration testing
- Violation handling: CI pipeline fails if direct console calls exceed threshold (more than 5 instances outside utilities)
- Violation handling: Code review feedback requests refactoring to use centralized logging utilities
- Violation handling: Documentation updates required if new logging patterns are introduced
- Exception process: Exceptions may be granted for third-party library integration code that requires specific console usage
- Exception process: Exception requests must document why centralized utilities cannot be used
- Exception process: Approved exceptions must be marked with inline comments explaining the deviation