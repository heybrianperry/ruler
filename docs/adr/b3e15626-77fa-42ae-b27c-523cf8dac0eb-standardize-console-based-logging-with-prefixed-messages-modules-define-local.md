# Standardize Console-Based Logging with Prefixed Messages: Modules Define Local

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses console-based logging (console.error, console.warn) across multiple modules including constants.ts, FileSystemUtils.ts, and CLI handlers
- Logging messages are consistently prefixed with identifiers such as '[ruler:verbose]', '[ruler]', and ERROR_PREFIX to distinguish message sources and severity levels
- The system operates in CLI and file system contexts where structured logging frameworks may introduce unnecessary dependencies, favoring lightweight console output
- Error handling spans file system operations, configuration directory checks, and CLI command validation, requiring consistent error reporting across different operational contexts

## Problem Statement

The application requires a consistent, lightweight logging mechanism that provides clear message attribution and severity levels across CLI operations, file system utilities, and core functionality without introducing external logging framework dependencies.

## Decision

1. MAY: Modules MAY define local prefix variables for context-specific logging when centralized prefixes are insufficient

## Policy Block

- MAY Modules MAY define local prefix variables for context-specific logging when centralized prefixes are insufficient

## Rationale

- Console-based logging provides zero-dependency observability suitable for CLI tools and file system utilities where external logging frameworks add unnecessary complexity
- Prefixed messages enable clear message attribution across distributed modules (constants.ts, FileSystemUtils.ts, CLI handlers) without requiring structured logging infrastructure
- The pattern appears consistently across 3 files with 90.33% confidence, indicating an established architectural convention rather than ad-hoc implementation
- Centralized logging functions (logVerbose, logInfo, logWarn) in constants.ts demonstrate intentional abstraction while maintaining console-based simplicity

## Consequences

Positive:
- Zero external dependencies for logging functionality, reducing bundle size and installation complexity
- Immediate visibility of log output in terminal environments without configuration or setup
- Clear message attribution through prefix conventions enables filtering and debugging across module boundaries
- Lightweight implementation suitable for CLI tools where startup time and resource usage are critical

Negative:
- Lack of structured logging limits machine-readable log parsing and integration with log aggregation systems
- No built-in log level filtering or runtime configuration without custom wrapper implementation
- Console output mixing with application stdout may complicate programmatic consumption of CLI tool output
- Limited observability features compared to dedicated logging frameworks (no timestamps, context injection, or transport flexibility)

## Alternatives

- Adopt a structured logging library (e.g., winston, pino, bunyan) with configurable transports and log levels (rejected)
  Rejected because: Introduces external dependencies and configuration complexity inappropriate for a lightweight CLI tool; console-based logging provides sufficient observability for the current operational context
  When valid: When log aggregation, structured querying, or multiple transport destinations become operational requirements
- Implement a custom Logger class with level filtering and configurable output streams (rejected)
  Rejected because: Adds implementation complexity without evidence of requirements for runtime log level configuration or stream redirection; current prefix-based approach meets observed needs
  When valid: When runtime log level control or testing scenarios require output stream mocking
- Use debug module for namespace-based debug logging with environment variable control (rejected)
  Rejected because: Requires external dependency and environment variable configuration; current implementation provides explicit logging functions (logVerbose, logInfo, logWarn) with clear semantics
  When valid: When fine-grained namespace-based debug control becomes necessary for troubleshooting complex module interactions

## Risks

- Console output may interfere with structured CLI output formats (JSON, machine-readable) if logging and application output are not properly separated
  Mitigation: Use stderr (console.error, console.warn) for all logging output and reserve stdout for application data; document output stream conventions
  Owner: engineering team
- Inconsistent prefix usage across modules may reduce log message clarity and complicate filtering
  Mitigation: Centralize prefix constants in constants.ts and enforce usage through code review; consider linting rules to detect direct console usage without prefixes
  Owner: engineering team
- Lack of log level filtering may produce excessive output in production environments or insufficient detail during debugging
  Mitigation: Implement environment variable or configuration-based log level control in centralized logging functions (logVerbose, logInfo, logWarn)
  Owner: engineering team

## Implementation Notes

- Define all logging prefix constants (ERROR_PREFIX, INFO_PREFIX, etc.) in src/constants.ts and export for module-wide reuse
- Implement centralized logging functions (logVerbose, logInfo, logWarn, logError) in constants.ts that encapsulate console methods and prefix application
- Use console.error for error and warning messages, console.log for informational output to maintain proper stderr/stdout separation
- Document logging conventions in developer guidelines including prefix format, severity level mapping, and when to use each logging function

## Continuation Context


Verify commands:
- grep -r "console\.\(error\|warn\|log\)" src/ | grep -v "node_modules" | head -20
- grep -r "\[ruler" src/ | grep -v "node_modules" | wc -l
- grep "export.*log" src/constants.ts

Accept when:
- All console logging calls include a prefix identifier indicating source or severity level
- Centralized logging functions (logVerbose, logInfo, logWarn) are exported from constants.ts and used consistently across modules
- Error and warning messages use console.error or console.warn to route output to stderr

## Enforcement

- Verified by: Code review verification that new logging calls use centralized functions or include appropriate prefixes
- Verified by: Grep-based verification commands in CI pipeline to detect unprefixed console usage
- Verified by: Manual testing of CLI output to confirm stderr/stdout separation
- Violation handling: Code review feedback requesting use of centralized logging functions
- Violation handling: CI pipeline warnings for direct console usage without prefixes
- Violation handling: Documentation updates to clarify logging conventions when violations indicate unclear guidelines
- Exception process: Direct console usage without prefixes may be approved for temporary debugging code not intended for merge
- Exception process: Exceptions for third-party library integration where console output cannot be controlled
- Exception process: Document approved exceptions in code comments with rationale and owner