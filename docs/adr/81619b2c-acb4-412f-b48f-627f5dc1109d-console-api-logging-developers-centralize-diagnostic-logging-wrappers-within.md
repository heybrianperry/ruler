# Console API Logging: Developers Centralize Diagnostic Logging Wrappers Within

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The application requires diagnostic and operational feedback to emit runtime warnings, error states, and verbose execution details across boundary layers.
- Standard runtime console streams provide direct output capabilities without requiring additional third-party dependencies or external telemetry infrastructure.
- Source modules across command processing, core filesystem interaction, and agent components directly invoke console error and warning functions with namespaced string prefixes.
- The absence of a centralized structured logging framework leaves log formatting, destination routing, and severity filtering distributed across individual call sites.

## Problem Statement

Diagnostic and operational error reporting across modules requires an output mechanism for terminal feedback and failure diagnostics. Without a dedicated logging framework, the application relies directly on runtime console output methods, which risks unstandardized formatting, uncoordinated log levels, and unmanageable output streams across application boundaries.

## Decision

1. SHOULD: Developers SHOULD centralize diagnostic logging wrappers within shared utility constants rather than scattering raw console invocations across domain logic.

## Policy Block

- SHOULD Developers SHOULD centralize diagnostic logging wrappers within shared utility constants rather than scattering raw console invocations across domain logic.

In scope:
- All application modules emitting diagnostic, warning, verbose, or error output to standard terminal streams.
- Command handling, filesystem operations, and agent execution layers requiring runtime operational status reporting.

Out of scope:
- Structured machine-to-machine data payloads returned as primary command outputs.
- External telemetry, metrics collection, and distributed tracing systems requiring remote transport protocols.

Exceptions:
- EXC-46-001: A module requires direct silent execution or suppressed terminal output for automated testing harnesses

## Rationale

- Direct usage of runtime console error and warning streams provides immediate diagnostic visibility without introducing external dependency overhead or binary bloat.
- Standardizing prefix conventions across console calls establishes baseline human readability for CLI diagnostics while remaining compatible with standard error redirection.
- Preserving native console logging provides zero-overhead telemetry suitable for lightweight utility execution where complex logging frameworks would add unnecessary runtime weight.

## Consequences

Positive:
- Zero third-party dependency footprint for operational and error logging.
- Immediate visibility of errors and warnings in terminal standard error streams.
- Low cognitive overhead and immediate developer familiarity with standard runtime console methods.

Negative:
- Lack of native structured log formats such as serialized JSON across output channels.
- No centralized dynamic log level configuration or runtime log filtering without manual wrapper logic.
- Potential risk of inconsistent prefixing and message formatting across distributed source files.

## Alternatives

- Adoption of an external structured logging framework (rejected)
  Rejected because: Introduces external dependencies, increased bundle footprint, and configuration complexity unnecessary for CLI utility operations
  When valid: Valid when multi-destination log streaming, remote log aggregation, or enterprise compliance audits are mandated
- Dedicated internal logging abstraction layer wrapping runtime console streams (deferred)
  Rejected because: Current codebase scope and operational requirements are adequately served by standard runtime console calls with prefix formatting
  When valid: Valid when multiple subsystems require uniform log level switching and pluggable transport sinks

## Risks

- Sensitive configuration values or environment credentials leaked through error console output
  Mitigation: Enforce error formatting utilities that sanitize input arguments and strip sensitive environment variables before logging
  Owner: Security Engineering
- Inconsistent error prefixing leading to confusion during CLI troubleshooting
  Mitigation: Define centralized prefix constants and verify compliance via static analysis and automated code review
  Owner: Core Engineering

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Format diagnostic errors using centralized error formatting helpers before passing messages to console error streams.
- Ensure standard output and standard error separation is maintained so diagnostic logs do not pollute command standard output.

## Continuation Context


Verify commands:
- discover_and_run_linter
- discover_and_run_test_suite

Accept when:
- Static analysis passes with zero unhandled or non-standard console errors across all source files.
- All automated unit and integration test suites pass without unexpected stderr emissions.

## Enforcement

- Verified by: Continuous integration automated linting and static analysis checks.
- Verified by: Peer code review verification against prefix conventions and error sanitization rules.
- Violation handling: Pull requests containing unapproved raw console calls or non-compliant output formatting will fail automated review checks.
- Violation handling: Violations identified during code review must be refactored to use standardized formatting helpers prior to merge.
- Exception process: Submit an architectural review request detailing the necessity for alternative logging channels or output suppression.
- Exception process: Exceptions must receive explicit sign-off from the technical lead and be documented in the corresponding module specification.