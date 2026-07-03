# Standardize Console-Based Logging for Error and Warning Events: Error Conditions File

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all agent implementations and core utility modules that handle error conditions, configuration failures, or file system operations requiring diagnostic output.

## Context

- The codebase includes agent implementations (FirebenderAgent) and core utilities (FileSystemUtils) that perform file system operations, configuration management, and external integration tasks where failures must be surfaced to operators or developers
- Error conditions such as JSON parsing failures, file read errors, and global config directory access issues require immediate visibility without introducing external logging dependencies
- The pattern emerged across 2 files with 90.50% confidence, indicating consistent adoption of console-based logging (console.warn, console.error) for diagnostic output in integration and configuration contexts
- The system processes environment variables (XDG_CONFIG_HOME), manages configuration files (firebender.json), and handles symbolic link validation, all requiring error transparency for debugging and operational awareness

## Problem Statement

Integration components and file system utilities need a lightweight, dependency-free mechanism to surface error and warning conditions during configuration loading, file parsing, and directory validation operations without coupling to external logging frameworks or introducing runtime overhead.

## Decision

1. MUST: Error conditions in file system operations, configuration parsing, and directory validation MUST be logged using console.error with descriptive context including the operation attempted and the error object

## Policy Block

- MUST Error conditions in file system operations, configuration parsing, and directory validation MUST be logged using console.error with descriptive context including the operation attempted and the error object

In scope:
- Agent implementations handling configuration and external integration
- Core file system utilities performing read, write, and validation operations
- Configuration loading and parsing logic
- Directory traversal and symbolic link validation
- Environment variable processing and global config directory access

Out of scope:
- High-frequency data processing operations
- Performance-critical runtime paths
- Production telemetry and metrics collection
- Structured log aggregation for centralized monitoring

Exceptions:
- EXC-001: A component requires structured logging with log levels, correlation IDs, or integration with external monitoring systems

## Rationale

- Console-based logging provides zero-dependency error visibility essential for debugging configuration failures and file system issues during development and operational troubleshooting
- The pattern appears consistently across agent and utility modules (2 files, 90.50% confidence), indicating organic adoption where simplicity and immediate feedback outweigh structured logging complexity
- File system operations and configuration parsing are inherently error-prone and benefit from immediate diagnostic output without requiring external infrastructure
- The approach aligns with Node.js conventions where console methods provide standard streams (stderr for errors) suitable for containerized and CLI environments

## Consequences

Positive:
- Zero external dependencies reduce bundle size and eliminate version conflicts with logging libraries
- Immediate error visibility in development and CLI contexts accelerates debugging of configuration and file system issues
- Standard console methods integrate naturally with container logging, process managers, and shell redirection
- Low cognitive overhead for developers familiar with JavaScript/TypeScript console API

Negative:
- Lack of structured logging limits integration with centralized log aggregation and monitoring platforms
- No built-in log levels, filtering, or conditional output beyond manual environment checks
- Console output cannot be easily disabled or redirected without process-level stream manipulation
- Missing correlation IDs and structured context complicates distributed tracing and multi-component debugging

## Alternatives

- Adopt a lightweight logging library (e.g., pino, winston) with structured output and configurable transports (rejected)
  Rejected because: Introduces external dependencies and configuration complexity disproportionate to current needs; evidence shows simple console logging suffices for file system and configuration error reporting in 2 observed modules
  When valid: When the system scales to require centralized log aggregation, correlation IDs, or integration with monitoring platforms
- Implement a custom logging abstraction wrapping console methods with configurable levels and formatting (rejected)
  Rejected because: Adds maintenance burden and abstraction overhead without evidence of need for log level filtering or custom formatting in current integration patterns
  When valid: When multiple components require consistent log formatting or runtime log level control becomes necessary
- Suppress error logging and rely on exception propagation to callers (rejected)
  Rejected because: Eliminates operational visibility into non-fatal errors and warnings; evidence shows console.warn usage for recoverable failures that should not propagate as exceptions
  When valid: Never for integration and file system utilities where diagnostic output is essential

## Risks

- Console logging in high-frequency operations could degrade performance or produce excessive output volume
  Mitigation: Enforce policy scope excluding performance-critical paths; conduct code review to identify inappropriate console usage in hot paths
  Owner: Engineering team
- Lack of structured logging may hinder future integration with centralized monitoring and alerting systems
  Mitigation: Document exception process for components requiring structured logging; plan migration path to logging framework when telemetry requirements emerge
  Owner: Architecture team
- Inconsistent log message formatting across modules complicates automated parsing and filtering
  Mitigation: Establish log message conventions (e.g., component prefixes, structured context) in implementation notes; provide examples in documentation
  Owner: Engineering team

## Implementation Notes

- Use console.error for unrecoverable errors and console.warn for recoverable failures or missing optional configuration; include the error object as the final argument to preserve stack traces
- Prefix log messages with component identifiers (e.g., '[ruler]', '[firebender]') to distinguish output sources in multi-component systems
- Include structured context in log messages: file paths, configuration keys, operation names, and relevant state to enable rapid diagnosis without additional tooling
- Avoid console logging in loops, recursive functions, or high-frequency event handlers; consider aggregating errors or using conditional logging based on environment variables for verbose output

## Continuation Context


Verify commands:
- grep -r 'console\.error\|console\.warn' src/ --include='*.ts' | grep -E '(FileSystemUtils|Agent)' | wc -l
- grep -r 'console\.log' src/core/FileSystemUtils.ts src/agents/FirebenderAgent.ts && echo 'FAIL: console.log found in integration modules' || echo 'PASS'
- npm test -- --grep 'logging' 2>&1 | grep -E '(error|warn)' | wc -l

Accept when:
- Verification commands confirm console.error and console.warn usage in FileSystemUtils and agent modules without console.log in production code paths
- Code review confirms error logs include error objects and sufficient context (file paths, operation names) for diagnosis
- No console logging appears in performance-critical paths or high-frequency operations identified during profiling

## Enforcement

- Verified by: Code review checklist requiring console.error/warn usage validation in integration and file system modules
- Verified by: Automated grep-based checks in CI pipeline detecting console.log usage in scoped modules
- Verified by: Periodic architecture review of logging patterns across new agent implementations
- Violation handling: Code review feedback requesting replacement of console.log with appropriate console.error or console.warn in integration contexts
- Violation handling: CI warnings for console logging in performance-critical paths identified by profiling
- Violation handling: Architecture review escalation for components requiring structured logging framework integration
- Exception process: Submit exception request documenting the need for structured logging, external monitoring integration, or alternative logging approach
- Exception process: Architecture team reviews exception against policy scope and approves logging framework choice
- Exception process: Document approved exception in component README with rationale and migration plan if applicable