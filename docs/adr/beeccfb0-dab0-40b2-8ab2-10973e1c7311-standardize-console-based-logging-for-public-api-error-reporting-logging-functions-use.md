# Standardize Console-Based Logging for Public API Error Reporting: Logging Functions Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses direct console methods (console.error, console.warn) for logging across public API surfaces in src/constants.ts, src/core/FileSystemUtils.ts, and src/cli/handlers.ts
- Logging is coupled with public contract exports (actionPrefix, createRulerError, logVerbose, logInfo, logWarn) and CLI error formatting functions
- Error reporting occurs at critical boundaries: file system operations checking global config directories, CLI handler validation preventing execution from .ruler directories, and verbose logging utilities
- The pattern emerged in a Node.js environment where process.env access and fs/path operations require observable error states for debugging and user feedback
- No structured logging framework or abstraction layer is present; console methods are invoked directly with string interpolation and prefix conventions

## Problem Statement

Public API surfaces and CLI handlers require consistent, observable error and warning reporting to support debugging, user feedback, and operational visibility, but the current implementation lacks a unified logging abstraction, making it difficult to control log levels, redirect output, or integrate with structured logging systems in production environments.

## Decision

1. MAY: Logging functions MAY use string interpolation or template literals for message composition to include dynamic runtime values

## Policy Block

- MAY Logging functions MAY use string interpolation or template literals for message composition to include dynamic runtime values

In scope:
- Public API exports in src/constants.ts (actionPrefix, createRulerError, logVerbose, logInfo, logWarn)
- File system utility functions in src/core/FileSystemUtils.ts that interact with global config directories
- CLI handler functions in src/cli/handlers.ts (applyHandler, initHandler, revertHandler)
- Error formatting and reporting functions exposed to end users

Out of scope:
- Internal debug logging not exposed through public APIs
- Third-party library logging behavior
- Test framework console output
- Development-only logging utilities

## Rationale

- Console-based logging provides immediate visibility in CLI environments and Node.js runtimes without requiring external dependencies or configuration
- The pattern supports three distinct severity levels (error, warn, verbose/info) that align with common operational needs for public APIs and CLI tools
- Prefix-based message formatting creates consistent, greppable log output that aids in debugging and operational monitoring
- Direct console usage maintains simplicity for a tool-focused codebase where structured logging infrastructure may be unnecessary overhead

## Consequences

Positive:
- Zero-dependency logging solution reduces package size and eliminates external logging framework vulnerabilities
- Consistent prefix conventions enable easy filtering and searching of log output in production environments
- Direct console methods provide predictable behavior across Node.js versions and runtime environments
- Exported logging functions (logVerbose, logInfo, logWarn) create a centralized point for future logging abstraction if needed

Negative:
- Direct console usage prevents runtime redirection of log output without monkey-patching global console object
- Lack of structured logging makes it difficult to integrate with log aggregation systems or observability platforms
- No built-in log level filtering mechanism forces verbose output to be controlled at call sites rather than centrally
- Testing code that uses console logging requires mocking global console methods, increasing test complexity

## Alternatives

- Adopt a structured logging library (e.g., winston, pino, bunyan) with configurable transports and log levels (rejected)
  Rejected because: Adds external dependency overhead and configuration complexity for a CLI tool where simple console output meets current observability requirements
  When valid: When integrating with centralized log aggregation systems or when structured JSON logging becomes a requirement
- Create a custom logging abstraction layer that wraps console methods with configurable output streams (deferred)
  Rejected because: Not rejected; deferred until evidence shows need for output redirection or testing isolation
  When valid: When testing requirements or production deployment scenarios require log output redirection without global console mocking
- Use Node.js built-in 'util.debuglog' for namespace-based debug logging (rejected)
  Rejected because: Requires NODE_DEBUG environment variable configuration and does not support warn/error severity levels needed for user-facing CLI output
  When valid: For internal debug logging that should be disabled by default and enabled only during development

## Risks

- Console output may be buffered or lost in containerized or serverless environments with non-standard stdout/stderr handling
  Mitigation: Document expected runtime environments and provide guidance for log capture in deployment documentation; consider adding flush mechanisms if deployment to such environments is planned
  Owner: engineering team
- Global console object modifications by third-party libraries could interfere with logging behavior or cause unexpected output formatting
  Mitigation: Capture references to console methods at module initialization time; monitor dependency updates for console-modifying behavior
  Owner: engineering team
- Lack of log level filtering may produce excessive output in production, impacting performance or overwhelming log storage
  Mitigation: Implement environment-based log level configuration through exported logging functions; provide clear documentation on controlling verbosity
  Owner: engineering team

## Implementation Notes

- Centralize logging function exports in src/constants.ts to provide a single import point for all logging operations
- Use consistent prefix patterns: '[ruler:verbose]' for verbose logs, '[ruler]' for standard logs, and ERROR_PREFIX constant for error messages
- Ensure CLI handlers format all user-facing errors through formatCliError before console output to maintain consistent error presentation
- Include contextual information (file paths, operation names, error objects) in error logs to support debugging without requiring verbose mode

## Continuation Context


Verify commands:
- grep -r 'console\.error\|console\.warn\|console\.log' src/ --include='*.ts' | grep -v 'logVerbose\|logInfo\|logWarn\|formatCliError'
- grep -E '(logVerbose|logInfo|logWarn|formatCliError)' src/constants.ts
- node -e "require('./dist/constants.js'); console.log('Logging exports validated')"

Accept when:
- All console.error and console.warn calls in public API files use consistent prefix formatting
- Exported logging functions (logVerbose, logInfo, logWarn) are present in src/constants.ts and used by CLI handlers
- CLI error messages are formatted through formatCliError or equivalent formatting function before console output

## Enforcement

- Verified by: Code review checklist requiring prefix consistency for all console logging calls
- Verified by: Grep-based verification commands in CI pipeline checking for direct console usage outside approved patterns
- Verified by: Manual testing of CLI error scenarios to verify consistent error message formatting
- Violation handling: CI pipeline fails if direct console calls are found outside of exported logging functions or formatCliError
- Violation handling: Code review blocks merge if error messages lack required prefix formatting or contextual information
- Violation handling: Documentation updates required for any new logging patterns introduced outside established conventions
- Exception process: Exception requests must document why standard logging functions are insufficient for the specific use case
- Exception process: Approval required from at least one maintainer familiar with the logging architecture
- Exception process: Approved exceptions must be documented in code comments explaining the rationale and alternative approach