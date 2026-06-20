# Standardize Structured Logging with Consistent Format and Levels: Log Messages Formatted

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all logging implementations across the codebase. All new logging code and modifications to existing logging code must comply with these rules.

## Context

- The codebase exhibits a consistent logging pattern across multiple modules (cli/handlers.ts, constants.ts, FileSystemUtils.ts), indicating an established logging architecture
- Observability requirements demand structured, consistent logging to enable effective debugging, monitoring, and operational support
- Multiple components require logging capabilities, suggesting a need for standardized logging practices to maintain consistency across the application
- The pattern appears in both CLI handlers and core utilities, indicating logging is a cross-cutting concern that spans different architectural layers

## Problem Statement

Without standardized logging practices, the codebase risks inconsistent log formats, varying log levels, and fragmented observability data that makes debugging and operational monitoring difficult. A unified logging approach is needed to ensure all components produce consistent, parseable, and actionable log output.

## Decision

1. SHOULD: Log messages SHOULD be formatted consistently to enable automated parsing and aggregation

## Policy Block

- SHOULD Log messages SHOULD be formatted consistently to enable automated parsing and aggregation

In scope:
- All TypeScript/JavaScript modules in src/ directory
- CLI handlers and command-line interface components
- Core utilities and file system operations
- Error handling and exception logging
- Operational events and state transitions

Out of scope:
- Third-party library internal logging
- Build-time or compile-time logging
- Test framework output and test-specific logging
- Development-only debug statements that are stripped in production builds

Exceptions:
- EXC-001: Emergency debugging requires temporary console.log statements
- EXC-002: Performance-critical paths where logging overhead is unacceptable

## Rationale

- Pattern detected across 3 files with 90.30% confidence indicates an established, intentional logging architecture rather than ad-hoc implementation
- Consistent logging patterns in both CLI handlers and core utilities demonstrate cross-cutting concern that benefits from standardization
- Structured logging enables better observability, faster debugging, and integration with monitoring and alerting systems
- Centralized logging approach reduces code duplication and ensures consistent behavior across all application components

## Consequences

Positive:
- Improved observability and operational visibility across all application components
- Easier debugging through consistent log formats and appropriate contextual information
- Better integration with log aggregation and monitoring tools
- Reduced cognitive load for developers who can rely on consistent logging patterns
- Enhanced security through standardized handling of sensitive data in logs

Negative:
- Initial refactoring effort required to standardize existing logging code
- Slight performance overhead from structured logging compared to simple console statements
- Learning curve for developers unfamiliar with the chosen logging framework
- Potential increase in log volume if not properly configured with appropriate levels

## Alternatives

- Use native console methods (console.log, console.error) throughout the codebase (rejected)
  Rejected because: Lacks structure, consistency, and advanced features like log levels, filtering, and metadata enrichment. Makes integration with monitoring tools difficult.
  When valid: Only appropriate for simple scripts or prototypes with no operational requirements
- Implement separate logging utilities per module without centralized framework (rejected)
  Rejected because: Creates inconsistency across modules, duplicates code, and makes it difficult to change logging behavior globally
  When valid: When modules have drastically different logging requirements that cannot be unified
- Adopt a comprehensive logging framework (e.g., Winston, Pino, Bunyan) (accepted)
  When valid: For production applications requiring robust observability and operational support

## Risks

- Excessive logging may impact application performance, especially in high-throughput scenarios
  Mitigation: Implement log level filtering, use asynchronous logging where possible, and establish guidelines for appropriate log verbosity
  Owner: Engineering team
- Sensitive data may inadvertently be logged, creating security and compliance issues
  Mitigation: Implement automated scanning for common sensitive patterns, provide developer training, and conduct code reviews focused on logging security
  Owner: Security team
- Log format changes may break existing log parsing and monitoring integrations
  Mitigation: Version log formats, provide migration guides, and maintain backward compatibility where possible
  Owner: DevOps team

## Implementation Notes

- Create a centralized logging utility module that wraps the chosen logging framework and provides a consistent API
- Define standard log levels and document when each level should be used (ERROR for failures, WARN for recoverable issues, INFO for significant events, DEBUG for detailed diagnostics)
- Establish naming conventions for logger instances (e.g., based on module or component name) to enable filtering and routing
- Configure log output format based on environment (structured JSON for production, human-readable for development)
- Implement log sanitization utilities to automatically redact sensitive data patterns before logging

## Continuation Context


Verify commands:
- grep -r 'console\.log\|console\.error' src/ --exclude-dir=node_modules --exclude='*.test.ts' | wc -l
- grep -r 'import.*logger\|from.*logger' src/ | wc -l
- npm run lint -- --rule 'no-console: error'

Accept when:
- Direct console.log/console.error usage in src/ is minimal or zero (excluding test files)
- Centralized logger import is present in all modules that perform logging
- Linting rules enforce logging standards and pass in CI pipeline

## Enforcement

- Verified by: ESLint rules configured to flag direct console usage (no-console rule)
- Verified by: Code review checklist includes verification of proper logging usage
- Verified by: Automated CI checks scan for sensitive data patterns in log statements
- Verified by: Static analysis tools verify logger imports and usage patterns
- Violation handling: CI pipeline fails if ESLint detects direct console usage in non-test code
- Violation handling: Code review requires changes before approval if logging standards are violated
- Violation handling: Security scanning flags potential sensitive data logging for immediate remediation
- Violation handling: Periodic audits identify and track logging standard compliance across the codebase
- Exception process: Developer documents exception rationale in code comments with ticket reference
- Exception process: Exception request submitted to architecture team for review and approval
- Exception process: Approved exceptions are documented in ADR amendments or exception registry
- Exception process: Exceptions are time-bound and reviewed quarterly for continued validity