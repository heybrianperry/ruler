# Console API Standard Stream Logging: Diagnostic Output Directed Standard Error Streams

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires operational diagnostic reporting across command-line interfaces and core utilities.
- Runtime console methods are currently used directly with custom string prefixes and error formatting functions.
- No external logging framework is present, keeping binary size minimal and avoiding additional dependency overhead.
- Inconsistent prefixing and direct stream writing across modules create a need for uniform diagnostic conventions.

## Problem Statement

Command-line handlers and core utility modules require a consistent, lightweight mechanism for logging operational warnings and errors without introducing heavy external logging dependencies or polluting standard output streams.

## Decision

1. MUST: Diagnostic output MUST be directed to standard error streams rather than standard output streams to keep data pipelines unpolluted.

## Policy Block

- MUST Diagnostic output MUST be directed to standard error streams rather than standard output streams to keep data pipelines unpolluted.

In scope:
- Diagnostic, warning, and error reporting across command-line interfaces and core modules.
- Operational failure reporting in local file system and configuration workflows.

Out of scope:
- Structured high-volume event streaming or distributed trace telemetry.
- Standard process output streams reserved for command payload data.

## Rationale

- Relying on built-in console error and warning APIs eliminates external dependency overhead and reduces maintenance burden.
- Standardizing prefixes across console output maintains readable diagnostic tracing for command-line users.
- Directing operational errors to standard error preserves the integrity of primary output streams in command-line environments.

## Consequences

Positive:
- Zero external dependency footprint for operational and diagnostic message handling.
- Immediate operational feedback delivered directly to standard error streams.
- Consistent error formatting across command execution boundaries.

Negative:
- Lack of structured logging schemas limits automated log indexing and search integration.
- No built-in log level filtering or dynamic log rotation without custom implementation.

## Alternatives

- Adopt an external structured logging framework (rejected)
  Rejected because: Introduces unnecessary dependency weight and configuration complexity for lightweight command-line diagnostic needs.
  When valid: Valid when application requirements expand to multi-channel log streaming, structured JSON output, or centralized aggregation services.
- Silent error handling without console emission (rejected)
  Rejected because: Obscures failure causes from operators and makes troubleshooting runtime issues difficult.
  When valid: Valid in headless embedded modes where errors are conveyed solely via return codes or serialized responses.

## Risks

- Unstructured console output may complicate automated error tracking in automated or continuous integration environments.
  Mitigation: Enforce consistent prefix formats and consider standard error capture utilities in verification scripts.
  Owner: engineering team
- Verbose console logging may expose internal path details or sensitive runtime information.
  Mitigation: Sanitize diagnostic output before passing strings to console error handlers.
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
- Route all failure details through designated error formatting helpers prior to emission.
- Ensure module prefixes are applied consistently across all console warning and error invocations.

## Continuation Context


Verify commands:
- Discover repository test and lint scripts from the project manifest and execute the static analysis suite.
- Execute the project verification script discovered from the build configuration to inspect logging stream compliance.

Accept when:
- Accept when all operational errors and warnings are emitted exclusively via console error and warning APIs with standardized prefixes.
- Accept when project verification and linting commands execute without logging-related violations.

## Enforcement

- Verified by: Static analysis and lint checks configured in the continuous integration pipeline.
- Verified by: Peer review of pull requests touching diagnostic logging and error handling.
- Violation handling: Pull requests containing unformatted or non-standard stream output must be updated before merging.
- Violation handling: Automated lint failures on unapproved logging calls block deployment.
- Exception process: Submit an architectural review request outlining why standard stream logging is insufficient for the specific component.
- Exception process: Document the rationale and obtain approval from the engineering team before introducing specialized logging mechanisms.