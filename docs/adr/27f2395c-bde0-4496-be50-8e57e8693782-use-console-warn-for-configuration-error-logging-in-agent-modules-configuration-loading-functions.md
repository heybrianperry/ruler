# Use console.warn for Configuration Error Logging in Agent Modules: Configuration Loading Functions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent modules that handle configuration loading and parsing operations.

## Context

- Agent modules require configuration loading from JSON files at initialization time, with potential for file system errors or malformed JSON content
- The FirebenderAgent module demonstrates a pattern of using console.warn for non-fatal configuration errors that should not halt agent initialization
- Configuration errors need visibility for debugging while allowing graceful degradation when existing configuration files are missing or corrupted
- The codebase uses Node.js fs and path modules for file system operations, requiring error handling for I/O and JSON parsing failures
- Agent modules expose public contracts (FirebenderAgent) that must remain operational even when configuration loading encounters errors

## Problem Statement

Agent modules need a consistent approach to logging configuration loading errors that provides operational visibility without treating recoverable errors as fatal failures, while maintaining clear diagnostic information for debugging configuration issues in production environments.

## Decision

1. MUST: Configuration loading functions MUST handle JSON.parse exceptions and log them without propagating to callers when graceful degradation is acceptable

## Policy Block

- MUST Configuration loading functions MUST handle JSON.parse exceptions and log them without propagating to callers when graceful degradation is acceptable

In scope:
- Agent modules that load configuration from JSON files (e.g., FirebenderAgent loading firebender.json)
- Configuration loading functions that use fs module for file I/O operations
- JSON.parse operations on configuration file content
- Initialization routines where configuration errors should not prevent agent startup

Out of scope:
- Fatal errors that should halt agent initialization
- Runtime errors unrelated to configuration loading
- Structured logging frameworks or third-party logging libraries
- Production logging infrastructure beyond console API

Exceptions:
- EXC-001: Configuration errors are critical to agent operation and must halt initialization
- EXC-002: A structured logging framework is adopted project-wide

## Rationale

- The evidence shows FirebenderAgent using console.warn with template literals to log configuration errors, establishing a pattern for non-fatal error visibility
- Using console.warn provides appropriate severity signaling (warning vs error) for recoverable configuration issues while maintaining stdout/stderr separation
- The pattern supports graceful degradation by allowing agents to initialize with default or fallback configuration when file loading fails
- Template literal error interpolation ensures diagnostic context is preserved without requiring structured logging infrastructure

## Consequences

Positive:
- Agent modules can start successfully even when configuration files are missing or corrupted, improving system resilience
- Developers receive clear diagnostic information about configuration issues through console output during development and in production logs
- Consistent warning-level logging across agent modules simplifies log filtering and monitoring
- Minimal dependencies on external logging frameworks reduce complexity and startup overhead

Negative:
- Console-based logging lacks structured metadata, making automated log analysis and alerting more difficult
- Warning messages may be overlooked in high-volume log streams without proper log level filtering
- No built-in log aggregation or correlation capabilities for tracking configuration issues across multiple agent instances
- Migration to structured logging frameworks requires refactoring all console.warn call sites

## Alternatives

- Throw exceptions for all configuration loading errors and require explicit error handling at call sites (rejected)
  Rejected because: Forces every agent initialization to implement try-catch blocks and prevents graceful degradation patterns that allow agents to start with default configuration
  When valid: When configuration is absolutely mandatory for agent operation and no sensible defaults exist
- Use a structured logging library (e.g., winston, pino) with warning level for configuration errors (deferred)
  Rejected because: Adds external dependencies and complexity; may be adopted project-wide in future but not justified for current scale
  When valid: When the project scales to require log aggregation, structured querying, or compliance with enterprise logging standards
- Silent failure with no logging, relying on default configuration behavior (rejected)
  Rejected because: Eliminates operational visibility into configuration issues, making debugging extremely difficult when agents behave unexpectedly due to missing configuration
  When valid: Never appropriate for configuration loading errors

## Risks

- Configuration errors may go unnoticed in production if warning logs are not monitored or filtered appropriately
  Mitigation: Establish log monitoring alerts for console.warn patterns related to configuration loading; document expected warning patterns in runbooks
  Owner: Operations team
- Inconsistent error message formats across different agent modules may complicate log parsing and analysis
  Mitigation: Provide template or example log message format in development guidelines; consider linting rules to enforce message structure
  Owner: Engineering team
- Console.warn may not be captured correctly in all deployment environments (e.g., containerized environments without stderr capture)
  Mitigation: Verify log capture configuration in deployment documentation; test warning visibility in staging environments
  Owner: DevOps team

## Implementation Notes

- Use template literals with error interpolation: console.warn(`Failed to read/parse existing firebender.json: ${error}`)
- Wrap JSON.parse calls in try-catch blocks within configuration loading functions (e.g., loadExistingConfig)
- Consider returning default configuration objects or null values after logging warnings to enable graceful degradation
- Document expected configuration file locations and formats in module README to help operators diagnose warning messages

## Continuation Context


Verify commands:
- grep -r "console\.warn.*Failed to read" src/agents/
- grep -r "JSON\.parse" src/agents/ | grep -A 5 "catch"
- npm test -- --grep "configuration.*error" 2>&1 | grep -i warn

Accept when:
- All agent modules that load configuration files use console.warn for non-fatal read/parse errors
- Error messages include file path and error details using template literals
- Configuration loading functions handle exceptions without propagating them when graceful degradation is implemented

## Enforcement

- Verified by: Code review checklist item for new agent modules verifying console.warn usage for configuration errors
- Verified by: Grep-based verification commands in CI pipeline checking for error handling patterns
- Verified by: Manual testing of agent initialization with missing or malformed configuration files
- Violation handling: Code review feedback requesting changes to use console.warn for configuration errors
- Violation handling: Documentation of violations in PR comments with reference to this ADR
- Violation handling: Refactoring tasks created for existing code that uses console.error or throws exceptions for non-fatal configuration errors
- Exception process: Developer documents why configuration error is fatal and cannot use graceful degradation
- Exception process: Module owner reviews and approves exception with justification in code comments
- Exception process: Exception documented in module README with explanation of mandatory configuration requirements