# Standardize console.error for Error Logging in Configuration and Environment Management: Error Messages Logged

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires error visibility during configuration directory resolution and CLI operations, particularly when accessing XDG_CONFIG_HOME and process environment variables
- File system operations involving symbolic link validation, directory traversal, and ruler directory detection need consistent error reporting to aid debugging
- CLI handlers must communicate error conditions to users when running from invalid locations or encountering configuration failures
- The runtime environment management layer interacts with OS-level paths (fs, path, os modules) and requires immediate error feedback without external logging dependencies

## Problem Statement

Configuration and environment management operations in src/core/FileSystemUtils.ts and src/cli/handlers.ts require a lightweight, dependency-free mechanism to report errors during runtime environment initialization, file system validation, and CLI execution, where structured logging frameworks may not yet be initialized or would introduce unnecessary complexity.

## Decision

1. MUST: Error messages logged via console.error MUST include contextual information such as file paths, directory names, or operation identifiers (e.g., '[ruler] Error checking global config directory')

## Policy Block

- MUST Error messages logged via console.error MUST include contextual information such as file paths, directory names, or operation identifiers (e.g., '[ruler] Error checking global config directory')

In scope:
- src/core/FileSystemUtils.ts error handling for directory validation and symbolic link checks
- src/cli/handlers.ts error reporting for CLI command validation and execution failures
- Runtime configuration source resolution involving process.env.XDG_CONFIG_HOME
- File system operations using fs, path, and os core modules within configuration management

Out of scope:
- Application-level business logic error handling outside configuration and environment management
- Structured logging for non-error informational or debug messages
- Error handling in modules where logging frameworks are already initialized and available
- Production telemetry and observability pipelines requiring structured log formats

## Rationale

- The pattern emerges from 2 files with 89.25% confidence, demonstrating consistent use of console.error in configuration and CLI layers where lightweight error reporting is essential
- Using console.error avoids circular dependencies and initialization ordering issues that would arise from importing structured logging frameworks during environment setup
- The obs.logging facet detection shows explicit console.error calls with contextual prefixes (e.g., '[ruler]', ERROR_PREFIX), indicating intentional standardization for user-facing error communication
- Core module dependencies (fs, path, os) and environment variable access (process.env.XDG_CONFIG_HOME) represent early-stage runtime operations where minimal dependencies are architecturally appropriate

## Consequences

Positive:
- Zero external dependencies for error logging during critical initialization phases reduces bootstrap complexity and failure modes
- Immediate error visibility in CLI and configuration contexts improves developer and user debugging experience
- Consistent error message formatting with contextual prefixes enables easier log parsing and troubleshooting
- Synchronous error output guarantees visibility even if the process terminates immediately after the error

Negative:
- console.error output lacks structured formatting, making automated log aggregation and analysis more difficult compared to JSON-based logging
- No built-in log levels, filtering, or conditional output mechanisms limit observability in production environments
- Error messages are written directly to stderr without buffering or batching, potentially causing performance issues under high error volumes
- Mixing console.error with structured logging in other modules creates inconsistent observability patterns across the codebase

## Alternatives

- Introduce a lightweight logging abstraction (e.g., pino, winston) for all error logging including configuration and environment management (rejected)
  Rejected because: Adds external dependencies and initialization complexity during bootstrap phase when configuration is still being resolved; creates circular dependency risks
  When valid: Valid for application-level modules after configuration and environment are fully initialized
- Implement a custom minimal logger class wrapping console.error with additional formatting and filtering capabilities (deferred)
  Rejected because: Adds abstraction overhead without clear evidence of need; current console.error usage with prefixes provides sufficient context
  When valid: Valid if error volume increases significantly or structured output becomes required for configuration errors
- Suppress error logging entirely and rely on thrown exceptions for error propagation (rejected)
  Rejected because: CLI tools require user-visible error messages; exceptions alone do not provide formatted, contextual feedback to end users
  When valid: Not applicable for user-facing CLI applications

## Risks

- Inconsistent error logging patterns emerge as the codebase grows, with some modules using console.error and others using structured logging frameworks
  Mitigation: Document clear boundaries: console.error for configuration/CLI initialization, structured logging for application logic; enforce via code review and linting rules
  Owner: engineering team
- Production environments may require structured log formats for aggregation, making console.error output difficult to parse and analyze at scale
  Mitigation: Implement stderr capture and transformation layer in deployment infrastructure; consider migration path to structured logging after initialization phase
  Owner: platform team
- Error messages lack internationalization support, limiting usability for non-English speaking users
  Mitigation: Defer i18n until user base requires it; maintain clear, simple English error messages with actionable guidance
  Owner: product team

## Implementation Notes

- Use consistent error message prefixes (e.g., '[ruler]', ERROR_PREFIX constants) to enable grep-based filtering and identification of error sources
- Include actionable context in error messages: file paths, directory names, operation types, and suggested remediation steps
- Ensure console.error calls occur before any process.exit() invocations to guarantee error visibility
- Document the boundary between console.error usage (configuration/CLI) and structured logging (application logic) in architecture guidelines

## Continuation Context


Verify commands:
- grep -r 'console\.error' src/core/FileSystemUtils.ts src/cli/handlers.ts | grep -E '(\[ruler\]|ERROR_PREFIX)'
- grep -r 'import.*logger' src/core/FileSystemUtils.ts src/cli/handlers.ts && exit 1 || exit 0
- node -e "require('./src/core/FileSystemUtils.ts'); require('./src/cli/handlers.ts')" 2>&1 | grep -q 'Error' && echo 'Error logging functional' || echo 'No errors detected'

Accept when:
- All error logging in src/core/FileSystemUtils.ts and src/cli/handlers.ts uses console.error with contextual prefixes
- No external logging library imports are present in configuration and environment management modules
- Error messages include sufficient context (paths, operation names) to enable debugging without additional tooling

## Enforcement

- Verified by: Code review checklist verifying console.error usage in configuration and CLI modules
- Verified by: Automated grep-based checks in CI pipeline detecting external logging imports in restricted modules
- Verified by: Manual testing of CLI error scenarios to validate error message visibility and formatting
- Violation handling: Pull requests introducing external logging dependencies in configuration/CLI modules are rejected with explanation
- Violation handling: Missing error context (paths, prefixes) triggers review comments requesting additional information
- Violation handling: Inconsistent error logging patterns are flagged during architecture review sessions
- Exception process: Exceptions require architectural review demonstrating that console.error is insufficient for the specific use case
- Exception process: Exception requests must document the initialization ordering and dependency implications of introducing logging frameworks
- Exception process: Approved exceptions are documented in ADR updates with rationale and scope limitations