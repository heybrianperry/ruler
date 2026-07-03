# Standardize Logging Patterns for CLI and Core Utilities: Core Utility Modules

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all logging implementations in CLI handlers, core utilities, and shared constants modules.

## Context

- The codebase exhibits a consistent logging pattern across CLI handlers (src/cli/handlers.ts), shared constants (src/constants.ts), and core file system utilities (src/core/FileSystemUtils.ts), indicating an established architectural approach to observability
- Logging infrastructure appears in public API contracts (facet: api.public.contracts), suggesting that logging behavior is part of the system's external interface and operational guarantees
- The pattern has been detected with 90.30% confidence across 3 files, indicating a deliberate and consistent implementation rather than ad-hoc logging
- CLI applications and file system utilities require structured logging for debugging, auditing, and operational monitoring, particularly for user-facing operations and system interactions
- The concentration of logging patterns in handlers and utilities suggests a layered approach where operational concerns are addressed at specific architectural boundaries

## Problem Statement

Without standardized logging patterns, the system risks inconsistent observability across different modules, making it difficult to trace operations, debug issues, and maintain operational visibility. The challenge is to establish clear logging conventions that work across CLI interfaces, core utilities, and shared configuration while maintaining consistency in log levels, formats, and contextual information.

## Decision

1. MUST: Core utility modules (e.g., FileSystemUtils) MUST log critical operations such as file I/O, permission checks, and error conditions with appropriate context

## Policy Block

- MUST Core utility modules (e.g., FileSystemUtils) MUST log critical operations such as file I/O, permission checks, and error conditions with appropriate context

In scope:
- CLI command handlers in src/cli/handlers.ts
- Core utility modules including FileSystemUtils
- Shared configuration and constants modules
- Public API contracts that expose operational behavior
- Error handling and exception logging across all layers

Out of scope:
- Third-party library logging configurations (unless wrapped by application code)
- Test fixtures and mock logging implementations
- Development-only debugging statements not intended for production
- Performance profiling or metrics collection (covered by separate observability concerns)

Exceptions:
- EXC-001: Temporary verbose logging is required for diagnosing production issues
- EXC-002: Legacy modules undergoing refactoring may temporarily use non-standard logging

## Rationale

- The pattern detection shows 90.30% confidence across 3 critical files (CLI handlers, constants, and core utilities), indicating this is an established and intentional architectural pattern rather than coincidental similarity
- Centralizing logging configuration in constants.ts enables consistent behavior across modules and simplifies maintenance when logging requirements change
- Implementing logging at the CLI handler and utility layer provides visibility at key architectural boundaries where user interactions and system operations occur
- The presence of logging in public API contracts (api.public.contracts facet) suggests that observable behavior is part of the system's operational contract with users and operators

## Consequences

Positive:
- Consistent logging patterns across CLI, utilities, and core modules improve debuggability and operational visibility
- Centralized logging configuration reduces duplication and makes it easier to adjust logging behavior system-wide
- Structured logging at architectural boundaries (handlers, utilities) provides clear audit trails for user operations and system interactions
- Standardized patterns reduce cognitive load for developers and make code reviews more effective

Negative:
- Additional logging infrastructure adds minor runtime overhead, though typically negligible for CLI applications
- Developers must learn and follow logging conventions, adding a small learning curve for new contributors
- Overly verbose logging can create noise and make it harder to identify critical issues if not properly managed with log levels
- Centralized configuration may create a single point of change that requires coordination across teams

## Alternatives

- Ad-hoc logging with no standardized patterns, allowing each module to implement logging independently (rejected)
  Rejected because: Leads to inconsistent observability, makes debugging difficult across module boundaries, and creates maintenance burden when logging requirements change
  When valid: Only appropriate for throwaway prototypes or single-file scripts with no operational requirements
- Comprehensive logging framework with aspect-oriented programming to automatically log all function entries/exits (rejected)
  Rejected because: Adds significant complexity and performance overhead, generates excessive log volume, and reduces developer control over what gets logged
  When valid: May be appropriate for highly regulated environments requiring complete audit trails of all operations
- Structured logging library (e.g., Winston, Pino) with JSON output for machine parsing (deferred)
  Rejected because: Not rejected, but deferred pending evaluation of operational requirements for log aggregation and analysis
  When valid: Should be reconsidered when deploying to production environments with centralized log management systems

## Risks

- Sensitive information (credentials, PII, tokens) may be inadvertently logged, creating security vulnerabilities
  Mitigation: Implement code review checklist for logging statements, use automated scanning for common sensitive patterns, and provide sanitization utilities for logging user input
  Owner: Engineering team with security review
- Excessive logging volume may impact performance or fill disk space in production environments
  Mitigation: Implement configurable log levels, use log rotation policies, and monitor log volume metrics with alerting thresholds
  Owner: Operations team
- Inconsistent adoption across the codebase may create observability gaps in critical paths
  Mitigation: Add linting rules to detect missing logging in handlers and utilities, include logging requirements in code review guidelines, and track coverage in architecture reviews
  Owner: Engineering team

## Implementation Notes

- Start by reviewing existing logging patterns in src/cli/handlers.ts, src/constants.ts, and src/core/FileSystemUtils.ts to understand current conventions
- Create or update centralized logging configuration in constants module with standard log levels, format specifications, and output destinations
- Implement logging wrappers or utilities that enforce consistent formatting and automatically include contextual information (timestamps, module names, operation IDs)
- Add logging to CLI handlers at operation boundaries: command start, validation steps, execution phases, and completion/error states
- Ensure FileSystemUtils and similar core utilities log all I/O operations, permission checks, and error conditions with sufficient context for debugging

## Continuation Context


Verify commands:
- grep -r "console\.log\|console\.error\|console\.warn" src/cli/handlers.ts src/core/FileSystemUtils.ts | wc -l
- grep -r "import.*logger\|import.*log" src/cli/handlers.ts src/constants.ts src/core/FileSystemUtils.ts
- find src -name '*.ts' -exec grep -l 'logging\|logger\|log\.' {} \; | grep -E '(handlers|constants|FileSystemUtils)'

Accept when:
- All CLI handlers in src/cli/handlers.ts contain logging statements for command execution, errors, and key operations
- Core utility modules like FileSystemUtils log critical operations (file I/O, errors) with appropriate context
- Logging configuration constants are centralized and referenced by handler and utility modules
- No sensitive information (credentials, tokens, PII) appears in log output during test execution

## Enforcement

- Verified by: Code review checklist requiring logging verification for CLI handlers and core utilities
- Verified by: Automated grep-based checks in CI pipeline to detect logging patterns in required modules
- Verified by: Architecture review of new modules to ensure logging standards are followed
- Violation handling: Pull requests missing required logging in handlers or utilities are flagged during code review
- Violation handling: CI pipeline warnings are generated for modules lacking logging patterns
- Violation handling: Technical debt tickets are created for existing modules that need logging improvements
- Exception process: Developer submits exception request with justification to engineering lead
- Exception process: Exception is reviewed for validity (e.g., legacy code, temporary debugging, performance-critical path)
- Exception process: Approved exceptions are documented in code with comments and tracked in technical debt backlog
- Exception process: Time-bound exceptions include removal date and follow-up ticket