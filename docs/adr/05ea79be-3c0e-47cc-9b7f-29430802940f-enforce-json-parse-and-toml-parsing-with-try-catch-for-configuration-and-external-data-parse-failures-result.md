# Enforce JSON.parse and TOML Parsing with Try-Catch for Configuration and External Data: Parse Failures Result

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is active for all agent implementations, configuration loaders, and modules that parse external data from JSON or TOML sources.

## Context

- The codebase processes configuration files and external data from multiple formats including JSON and TOML across agent implementations (FirebenderAgent, MistralVibeAgent, GeminiCliAgent), configuration loaders (ConfigLoader, UnifiedConfigLoader), MCP path handlers, and VSCode settings managers
- Seven files demonstrate consistent use of JSON.parse and parseTOML operations on file system content, environment variables, and external configuration sources without guaranteed schema validation at parse time
- Agent configuration systems read and write JSON files (firebender.json, gemini-cli.json) and TOML files (mistral-vibe.toml) from disk, requiring resilient parsing to handle malformed, missing, or corrupted files
- The UnifiedConfigLoader and ConfigLoader modules use Zod schemas for validation but only after successful parsing, creating a gap where parse failures must be handled separately from validation failures
- Console.warn logging in FirebenderAgent demonstrates awareness of parse failure scenarios, indicating that error handling for parsing operations is a recognized concern in the architecture

## Problem Statement

Parsing untrusted or external data from JSON and TOML sources without proper error handling exposes the system to runtime crashes, information disclosure through stack traces, and denial of service when malformed input is encountered. The current architecture requires consistent error handling patterns across all parsing operations to maintain system stability and security.

## Decision

1. MUST: Parse failures MUST result in either graceful degradation with default values, explicit error propagation to the caller, or controlled application termination with clear error messages

## Policy Block

- MUST Parse failures MUST result in either graceful degradation with default values, explicit error propagation to the caller, or controlled application termination with clear error messages

In scope:
- All agent implementations that read configuration files (FirebenderAgent, MistralVibeAgent, GeminiCliAgent, AgentsMdAgent)
- Configuration loading modules (ConfigLoader, UnifiedConfigLoader)
- MCP path handlers and VSCode settings managers that parse JSON
- Any module that calls JSON.parse or parseTOML on data originating from file system, environment variables, or external sources
- RuleProcessor and related modules that process external configuration

Out of scope:
- Parsing of hardcoded JSON literals in source code
- Parsing of data generated internally by the application in the same execution context
- Test fixtures and mock data in unit tests (though error handling tests are encouraged)
- JSON.stringify operations (encoding rather than parsing)

Exceptions:
- EXC-001: Parsing occurs within a higher-level error boundary that already catches and handles all exceptions from the parsing operation
- EXC-002: The module is explicitly designed to crash on parse failure as part of fail-fast initialization (e.g., critical bootstrap configuration)

## Rationale

- The evidence shows 7 files with 91.07% confidence performing JSON.parse and parseTOML operations on external data, establishing this as a consistent architectural pattern requiring standardized error handling
- FirebenderAgent's existing console.warn pattern for parse failures demonstrates that the codebase already recognizes the need for resilient parsing, and this ADR formalizes that practice across all modules
- Separation of parse errors from Zod validation errors (as seen in ConfigLoader and UnifiedConfigLoader) enables more precise error messages and better debugging experience for users encountering configuration issues
- The api.public.contracts facet indicates these modules expose public interfaces, making robust error handling critical for preventing cascading failures in dependent code

## Consequences

Positive:
- Prevents application crashes from malformed JSON or TOML files, improving system reliability and user experience
- Enables graceful degradation and recovery strategies when configuration files are corrupted or incomplete
- Provides clear, actionable error messages to users and operators when parse failures occur, reducing time to resolution
- Reduces security risk of information disclosure through unhandled exception stack traces that might reveal internal paths or data structures

Negative:
- Increases code verbosity with try-catch blocks around every external parse operation
- Requires developers to make explicit decisions about error handling strategy (fail-fast vs. graceful degradation) for each parse operation
- May mask underlying issues if error handling defaults to silent failures or overly permissive fallbacks
- Adds cognitive overhead for code reviewers to verify that error handling is appropriate for each context

## Alternatives

- Use a centralized parsing utility that wraps JSON.parse and parseTOML with standard error handling (deferred)
  When valid: Could be adopted as an implementation strategy after this ADR is accepted, providing a consistent API while enforcing the error handling requirements
- Rely on Zod schema validation alone without explicit parse error handling (rejected)
  Rejected because: Zod validation occurs after parsing, so parse failures would still cause unhandled exceptions. The evidence shows ConfigLoader and UnifiedConfigLoader already use Zod but still need parse-level error handling
  When valid: Not applicable - parse errors and validation errors are distinct failure modes requiring separate handling
- Use JSON5 or other permissive parsers that tolerate malformed input (rejected)
  Rejected because: Permissive parsing can mask configuration errors and lead to unexpected behavior. The evidence shows the codebase uses standard JSON.parse and @iarna/toml, indicating a preference for strict parsing with explicit error handling
  When valid: Could be considered for specific user-facing configuration files where comments and trailing commas improve usability, but would still require try-catch for other error conditions

## Risks

- Inconsistent error handling strategies across modules could lead to confusing user experience where some parse failures crash the application and others fail silently
  Mitigation: Establish clear guidelines in implementation notes for when to use fail-fast vs. graceful degradation. Code review checklist should verify error handling strategy is appropriate for the module's role (critical bootstrap vs. optional feature)
  Owner: Engineering team and code reviewers
- Overly broad try-catch blocks might catch and suppress errors beyond parse failures, hiding bugs in surrounding code
  Mitigation: Limit try-catch scope to only the parse operation and immediate result handling. Verify caught exceptions are of expected types (SyntaxError for JSON.parse). Include this in code review guidelines
  Owner: Engineering team
- Legacy code and new contributions may not adopt the error handling pattern without enforcement mechanisms
  Mitigation: Implement linting rules or static analysis to detect unprotected JSON.parse and parseTOML calls. Add verification commands to CI pipeline. Provide code snippets and templates in developer documentation
  Owner: DevOps and engineering team

## Implementation Notes

- For agent initialization code (FirebenderAgent, MistralVibeAgent, GeminiCliAgent), use console.warn for parse failures and continue with default configuration to allow the agent to start even if custom config is malformed
- For critical configuration loaders (ConfigLoader, UnifiedConfigLoader), propagate parse errors to the caller with context about which file failed, allowing the application to decide whether to fail-fast or fall back to defaults
- When logging parse errors, include the file path and operation type but avoid logging the raw malformed content which might contain sensitive data or be excessively large
- Consider implementing a parseJSON utility function that standardizes error handling and logging, reducing boilerplate while ensuring consistent behavior across the codebase

## Continuation Context


Verify commands:
- grep -r 'JSON\.parse' src/ | grep -v 'try' | grep -v 'catch' | grep -v '//' | grep -v test
- grep -r 'parseTOML' src/ | grep -v 'try' | grep -v 'catch' | grep -v '//' | grep -v test
- npm test -- --grep 'parse.*error|malformed.*json|invalid.*toml'

Accept when:
- All JSON.parse and parseTOML calls on external data sources in src/ are wrapped in try-catch blocks or documented error boundaries
- Parse error handlers log appropriate context and either propagate errors, use defaults, or terminate gracefully
- Test coverage includes malformed JSON and TOML input scenarios for all configuration loaders and agent implementations

## Enforcement

- Verified by: Code review checklist item for all PRs touching configuration loading or external data parsing
- Verified by: Static analysis or linting rules to detect unprotected JSON.parse and parseTOML calls
- Verified by: CI pipeline verification commands checking for unprotected parse operations
- Violation handling: PR review feedback requesting addition of try-catch blocks or documentation of error boundary
- Violation handling: CI pipeline warnings or failures when verification commands detect unprotected parse operations
- Violation handling: Post-merge remediation for violations discovered in production through incident review
- Exception process: Developer documents the exception case (EXC-001 or EXC-002) in code comments with rationale
- Exception process: Code reviewer verifies the exception is valid and the documentation is sufficient
- Exception process: Architecture review required for new exception categories not covered by EXC-001 or EXC-002