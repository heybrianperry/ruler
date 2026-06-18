# Standardize console-based logging for warnings and errors across core modules: Modules Define Custom

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all modules in the Frameworks & Libraries > Libraries & Modules category that perform observability logging operations.

## Context

- The codebase exhibits a consistent pattern of using console.warn and console.error for diagnostic output across core modules including FirebenderAgent, ConfigLoader, handlers, and FileSystemUtils
- Configuration loading, file system operations, and agent initialization require observable error handling to support debugging and operational visibility
- The system processes configuration from multiple sources (XDG_CONFIG_HOME, process.env) and performs file I/O operations that may fail, requiring standardized error reporting
- Four core modules demonstrate convergent adoption of console-based logging without a centralized logging abstraction, indicating an emergent architectural pattern

## Problem Statement

Core modules require a consistent approach to logging warnings and errors during configuration loading, file system operations, and agent initialization to ensure operational visibility and debuggability without introducing external logging dependencies.

## Decision

1. MAY: Modules MAY define custom error prefixes or formatting conventions to maintain consistency within their functional domain

## Policy Block

- MAY Modules MAY define custom error prefixes or formatting conventions to maintain consistency within their functional domain

In scope:
- Configuration loading operations (ConfigLoader.ts)
- File system utilities (FileSystemUtils.ts)
- Agent initialization and configuration (FirebenderAgent.ts)
- CLI command handlers (handlers.ts)
- Error conditions during JSON.parse, file read/write, and directory traversal operations

Out of scope:
- Application-level logging in user-facing applications that consume these libraries
- Structured logging for production telemetry or monitoring systems
- Debug-level logging for verbose diagnostic output
- Performance metrics or tracing instrumentation

Exceptions:
- EX-001: A module requires structured logging for integration with external monitoring systems

## Rationale

- Evidence from 4 files shows consistent adoption of console.warn and console.error across core modules with 90.03% confidence, indicating an established architectural pattern
- Console-based logging provides zero-dependency observability suitable for library code that must remain lightweight and avoid coupling to specific logging frameworks
- The pattern supports debugging of configuration loading (XDG_CONFIG_HOME, process.env), file I/O failures, and JSON parsing errors which are critical failure modes in the observed modules
- Standardizing on console methods enables consistent error reporting while allowing consuming applications to intercept or redirect output as needed

## Consequences

Positive:
- Zero external dependencies for logging reduces bundle size and eliminates version conflicts in consuming applications
- Console methods are universally available in Node.js and browser environments, maximizing portability
- Consistent error reporting across core modules improves debuggability and operational visibility
- Consuming applications retain full control over log interception, formatting, and routing through console method overrides

Negative:
- Console-based logging lacks structured output (JSON), log levels, or metadata that production monitoring systems typically require
- No built-in log filtering, sampling, or rate limiting capabilities for high-volume scenarios
- Error context is limited to string formatting rather than structured fields, making automated log parsing more difficult
- Testing code that logs to console requires mocking or capturing console methods, adding test complexity

## Alternatives

- Adopt a lightweight logging framework (e.g., pino, winston) as a standard dependency across core modules (rejected)
  Rejected because: Introduces external dependencies that increase bundle size and create coupling, contradicting the library's goal of remaining lightweight and dependency-minimal
  When valid: When building application-level services rather than reusable library modules, or when structured logging is a hard requirement
- Implement a custom minimal logging abstraction with pluggable backends (rejected)
  Rejected because: Adds complexity and maintenance burden for functionality that console methods already provide adequately for library-level error reporting
  When valid: If the codebase grows to require log level filtering, structured output, or multiple output targets within the library itself
- Suppress all logging and rely on thrown exceptions for error communication (rejected)
  Rejected because: Eliminates visibility into non-fatal warnings and fallback behaviors that are important for debugging configuration issues and file system operations
  When valid: For pure library functions where all error conditions should be propagated to callers rather than logged

## Risks

- Inconsistent log message formatting across modules makes automated parsing and filtering difficult
  Mitigation: Establish and document log message format conventions (prefix patterns, context inclusion) and enforce through code review
  Owner: Engineering team
- Console logging in library code may produce unwanted output in consuming applications that expect silent operation
  Mitigation: Document logging behavior in module README and provide guidance for consumers to intercept console methods if needed
  Owner: Engineering team
- Lack of structured logging limits integration with production monitoring and alerting systems
  Mitigation: Accept this limitation for library code; consuming applications should wrap or augment logging as needed for their monitoring requirements
  Owner: Engineering team

## Implementation Notes

- Use console.error for all error conditions that indicate failure or unexpected state; include the operation name, affected resource (file path, config key), and error message
- Use console.warn for non-fatal conditions such as missing optional configuration, fallback behaviors, or deprecated usage patterns
- Include module-specific prefixes (e.g., '[ruler]', ERROR_PREFIX constant) in log messages to enable filtering and identification in multi-module applications
- When logging errors from caught exceptions, include both the operation context and the original error message: console.error(`Operation failed: ${err.message}`)

## Continuation Context


Verify commands:
- grep -r 'console\.error' src/ | grep -E '(ConfigLoader|FileSystemUtils|FirebenderAgent|handlers)' | wc -l
- grep -r 'console\.warn' src/ | grep -E '(ConfigLoader|FileSystemUtils|FirebenderAgent|handlers)' | wc -l
- ! grep -r 'import.*winston\|import.*pino\|import.*bunyan' src/core/ src/agents/

Accept when:
- All error conditions in ConfigLoader, FileSystemUtils, FirebenderAgent, and handlers modules are logged using console.error with descriptive context
- All warning conditions use console.warn with sufficient context to understand the non-fatal issue
- No external logging framework dependencies are introduced in core library modules (src/core/, src/agents/)

## Enforcement

- Verified by: Code review verification that error handling uses console.error/console.warn with appropriate context
- Verified by: Automated grep-based checks in CI to detect external logging framework imports in core modules
- Verified by: Manual testing of error scenarios to verify log output includes operation context and error details
- Violation handling: Code review feedback requesting use of console methods instead of external logging frameworks
- Violation handling: CI pipeline warnings if external logging dependencies are detected in core library modules
- Violation handling: Refactoring requests for log messages that lack sufficient context (operation, resource, error message)
- Exception process: Submit architecture review request documenting the specific requirement for structured logging or external framework
- Exception process: Provide justification for bundle size impact and dependency coupling
- Exception process: Document the exception and integration requirements in module README and ADR amendments