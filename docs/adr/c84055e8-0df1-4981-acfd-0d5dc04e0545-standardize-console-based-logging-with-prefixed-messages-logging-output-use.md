# Standardize Console-Based Logging with Prefixed Messages: Logging Output Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase uses console-based logging (console.error, console.warn) across multiple modules including constants.ts, FileSystemUtils.ts, and handlers.ts
- Logging statements consistently employ prefixed message patterns such as `[ruler:verbose]`, `[ruler]`, and `${ERROR_PREFIX}` to provide context and categorization
- The system requires operational visibility during file system operations, configuration resolution, and CLI command execution
- No structured logging framework or external logging library is detected in the evidence; logging relies on native console APIs

## Problem Statement

The application needs a consistent approach to operational logging that provides context, severity levels, and traceability without introducing external dependencies, while supporting both verbose diagnostic output and standard error reporting across file system utilities, CLI handlers, and core constants.

## Decision

1. MUST: All logging output MUST use native console APIs (console.error, console.warn, console.log) without external logging frameworks

## Policy Block

- MUST All logging output MUST use native console APIs (console.error, console.warn, console.log) without external logging frameworks

In scope:
- All modules performing file system operations (FileSystemUtils.ts)
- CLI command handlers (handlers.ts)
- Core utility and constant definitions (constants.ts)
- Error reporting and diagnostic output

Out of scope:
- Third-party library internal logging
- Browser-based console usage outside the Node.js runtime
- Structured log aggregation or external log shipping

## Rationale

- The pattern is observed across 3 files with 90.33% confidence, indicating consistent adoption of console-based logging with prefixed messages
- Using native console APIs eliminates external dependencies while providing sufficient operational visibility for a CLI tool
- Prefixed messages enable log filtering, categorization, and severity identification without structured logging infrastructure
- Centralized logging functions (logVerbose, logInfo, logWarn) in constants.ts establish a public API contract for consistent usage

## Consequences

Positive:
- Zero external dependencies for logging functionality, reducing bundle size and maintenance burden
- Consistent message prefixes enable easy filtering and parsing of log output in CI/CD pipelines
- Native console APIs provide immediate output to stderr/stdout without buffering or configuration overhead
- Centralized logging functions in constants.ts create a single point of control for logging behavior

Negative:
- Console-based logging lacks structured metadata (timestamps, log levels, context objects) available in dedicated logging frameworks
- No built-in support for log level filtering at runtime without modifying code
- Difficult to redirect or aggregate logs to external monitoring systems without wrapping console methods
- Prefix-based categorization is less robust than structured logging with fields and tags

## Alternatives

- Adopt a structured logging library (e.g., winston, pino, bunyan) (rejected)
  Rejected because: Introduces external dependencies and configuration complexity not justified by current operational requirements; console-based logging is sufficient for CLI tool diagnostics
  When valid: When log aggregation, remote shipping, or complex filtering requirements emerge
- Implement a custom logging abstraction layer with configurable transports (rejected)
  Rejected because: Over-engineering for current needs; adds maintenance burden without clear operational benefit given the simplicity of console APIs
  When valid: When multiple output targets (file, network, console) or runtime log level configuration becomes necessary
- Use debug module for namespaced debug logging (deferred)
  Rejected because: Not currently adopted in evidence; could complement existing console logging for verbose diagnostics
  When valid: When fine-grained debug namespace control is needed for troubleshooting complex interactions

## Risks

- Inconsistent prefix usage across modules leads to fragmented log output that is difficult to filter or parse
  Mitigation: Enforce use of centralized logging functions (logVerbose, logInfo, logWarn) from constants.ts; document prefix conventions in code review guidelines
  Owner: engineering team
- Console-based logging may not scale to production monitoring requirements if the tool evolves beyond CLI usage
  Mitigation: Monitor operational requirements; plan migration to structured logging if log aggregation or remote monitoring becomes necessary
  Owner: engineering team
- Lack of log level filtering at runtime forces verbose output to be always-on or always-off at code level
  Mitigation: Implement environment variable-based log level control (e.g., RULER_LOG_LEVEL) if runtime filtering becomes necessary
  Owner: engineering team

## Implementation Notes

- Import logging functions (logVerbose, logInfo, logWarn) from constants.ts rather than calling console methods directly
- Use template literals with prefix constants for consistent message formatting: `${prefix} ${message}`
- Reserve console.error for error conditions, console.warn for warnings, and verbose logging for diagnostic output
- Document expected log prefixes ([ruler], [ruler:verbose], ERROR_PREFIX) in module-level comments for maintainability

## Continuation Context


Verify commands:
- grep -r "console\.error\|console\.warn\|console\.log" src/ --include="*.ts" | grep -E "\[ruler|ERROR_PREFIX|\$\{prefix\}"
- grep -r "logVerbose\|logInfo\|logWarn" src/ --include="*.ts"
- grep "export.*log" src/constants.ts

Accept when:
- All console logging calls in src/ include a descriptive prefix (e.g., [ruler], [ruler:verbose], ${ERROR_PREFIX})
- Centralized logging functions (logVerbose, logInfo, logWarn) are exported from constants.ts and used consistently
- No direct console calls exist without prefix formatting in core modules (constants.ts, FileSystemUtils.ts, handlers.ts)

## Enforcement

- Verified by: Code review verification of logging call patterns and prefix usage
- Verified by: Grep-based verification commands in CI pipeline checking for prefixed console calls
- Verified by: Manual inspection of log output during integration testing
- Violation handling: Code review feedback requesting use of centralized logging functions from constants.ts
- Violation handling: CI pipeline warnings for unprefixed console calls in core modules
- Violation handling: Refactoring tasks created for modules with inconsistent logging patterns
- Exception process: Document rationale for direct console usage in code comments
- Exception process: Obtain approval from code reviewer for exceptional cases (e.g., third-party integration constraints)
- Exception process: Record exception in module documentation with justification