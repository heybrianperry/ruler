# formatCliError Formatted CLI Error Logging: Cli Handlers Normalize Failure Strings Via

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Command-line execution requires uniform presentation of error diagnostics to terminal users without exposing internal process traces.
- Handler modules intercept operational failures and convert them into human-readable terminal alerts before terminating or recovering.
- The codebase routes error notifications through formatCliError prior to dispatching them to standard console error streams.

## Problem Statement

Unformatted error output in command-line handlers risks presenting inconsistent diagnostic messages to users and leaking internal stack traces or unformatted strings into standard error streams.

## Decision

1. SHOULD: CLI handlers SHOULD normalize all failure strings via formatCliError before returning exit codes or completing error transitions.

## Policy Block

- SHOULD CLI handlers SHOULD normalize all failure strings via formatCliError before returning exit codes or completing error transitions.

In scope:
- All command-line interface execution handlers managing user-facing command dispatch and failure responses.

Out of scope:
- Internal core domain logic, lower-level filesystem utilities, and non-CLI background operations.

Exceptions:
- EXC-46-001: Low-level system diagnostics require raw binary or unformatted stream redirection during emergency panic recovery.

## Rationale

- Pre-formatting messages with formatCliError ensures consistent visual structure for terminal errors across CLI handler invocations.
- Directing formatted output to console.error ensures that failure reporting is properly isolated to standard error rather than standard output streams.
- A centralized formatting call prevents individual command handlers from introducing divergent message conventions.

## Consequences

Positive:
- Standardizes terminal error formatting across command-line handler operations.
- Ensures clean separation between standard output and standard error output streams.
- Prevents raw internal error representations from being dumped directly to terminal sessions.

Negative:
- Couples command error presentation directly to standard console error streams without structured log aggregation.
- Requires every command handler to explicitly invoke formatting wrappers prior to logging.

## Alternatives

- Direct unformatted console error invocations across all command handlers (rejected)
  Rejected because: Leads to inconsistent terminal error formats and potential leakage of unhandled stack traces.
  When valid: Valid during early prototyping before message normalization is established.
- Structured machine-readable log output across terminal interfaces (rejected)
  Rejected because: Degrades readability and usability for interactive human terminal operators.
  When valid: Valid in daemonized background services or non-interactive containerized environments.

## Risks

- Handlers may bypass formatCliError and call standard output or error mechanisms directly.
  Mitigation: Establish automated linting rules to enforce error message wrapping before console error dispatch.
  Owner: engineering team
- Sensitive configuration values may inadvertently be included in formatted error messages.
  Mitigation: Sanitize configuration inputs before passing error strings to formatting routines.
  Owner: engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Verify that all user-facing command dispatch branches catch thrown errors and pass the message string through formatCliError.
- Ensure standard error emissions remain decoupled from standard result output pipelines.

## Continuation Context


Verify commands:
- Discover and run the repository linting suite to verify compliance with error formatting rules across CLI handlers.
- Discover and execute the automated test suite targeting CLI error handling and terminal output streams.

Accept when:
- All command-line error handlers process failure strings through formatCliError before calling console.error.
- Automated tests confirm error output is written to standard error with formatted structure.

## Enforcement

- Verified by: Automated static analysis and code review checks during pull request evaluation.
- Verified by: Automated integration tests validating terminal error emission behavior.
- Violation handling: Pull requests containing unformatted error calls to console streams will fail code review and static analysis gates.
- Exception process: Exceptions must be submitted via architectural review request and documented with operational justification.