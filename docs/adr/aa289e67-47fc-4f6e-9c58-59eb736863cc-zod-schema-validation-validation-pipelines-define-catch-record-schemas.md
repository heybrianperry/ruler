# Zod Schema Validation: Validation Pipelines Define Catch Record Schemas

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Configuration loading and external agent integrations parse external input artifacts to construct internal runtime definitions.
- Schema-level validation using Zod is observed in the primary configuration loader with strict object constraints and nested record shapes.
- Peripheral modules and agent adapters primarily perform raw object deserialization without enforcing declarative schema contracts.
- The absence of a uniform cross-boundary validation framework across all integration boundaries leaves input validation decentralized.

## Problem Statement

External configuration artifacts and agent definitions enter the system through disparate file parsers and deserialization calls, risking runtime type errors and unvalidated state propagation when input structures deviate from expected schemas.

## Decision

1. MAY: Validation pipelines MAY define catch-all record schemas for arbitrary key-value mapping blocks when configuration keys cannot be predetermined.

## Policy Block

- MAY Validation pipelines MAY define catch-all record schemas for arbitrary key-value mapping blocks when configuration keys cannot be predetermined.

In scope:
- Parsing and loading of persistent configuration data from environment paths and storage locations.
- Construction of internal runtime models from external serialized payloads.

Out of scope:
- Internal memory-to-memory transfers between trusted core abstractions.
- Low-level stream read and write operations prior to structural interpretation.

Exceptions:
- EX-42-001: Parsing legacy configuration blocks where schema structures cannot be statically defined and dynamic fallbacks are required.

## Rationale

- Zod provides declarative runtime validation that ensures parsed configuration data conforms to expected application contracts at initialization time.
- Strict schema evaluation rejects unexpected properties, preventing silent configuration drift and misconfiguration bugs.
- Limiting schema validation to configuration ingestion currently avoids overhead in high-throughput data paths while securing primary system inputs.

## Consequences

Positive:
- Malformed configuration inputs fail fast at startup with descriptive validation diagnostics.
- Type inference derived directly from schemas keeps static interface definitions synchronized with runtime validations.
- Strict schema evaluation mitigates unexpected property injection.

Negative:
- Additional schema maintenance is required when configuration shapes evolve.
- Validation execution introduces runtime evaluation overhead during application initialization.
- Incomplete codebase adoption creates asymmetric security guarantees across different modules.

## Alternatives

- Ad-hoc manual type guarding and property presence checks following deserialization (rejected)
  Rejected because: Manual type checks are error-prone, verbose, and difficult to keep synchronized with evolving type definitions.
  When valid: Simple scripts with minimal input structures that do not justify schema dependencies.
- Unchecked type casting directly following parsing (rejected)
  Rejected because: Unchecked casting bypasses runtime security guarantees and allows corrupt data to propagate silently into core systems.
  When valid: Never valid for external untrusted payloads.

## Risks

- Breaking changes in external input shapes cause startup crashes when strict validation rejects unrecognized fields.
  Mitigation: Incorporate schema evolution strategies with optional fields and deprecation fallbacks.
  Owner: engineering team
- Discrepancy between validated configuration loading and unvalidated agent modules creates inconsistent input security posture.
  Mitigation: Progressively expand schema validation to all external deserialization entry points.
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
- Define reusable schema fragments for common structures such as server records and feature flags to maintain consistency across configuration schemas.
- Use strict parsing modes on root configuration objects while using optional catch-all handlers on extensible sub-blocks.

## Continuation Context


Verify commands:
- Discover and execute the project unit test suite for configuration and schema validation modules.
- Run the repository static analysis and type checking scripts to verify that schema types align with declared contracts.

Accept when:
- The configuration validation test suite passes with complete verification of valid, invalid, and legacy input shapes.
- All external configuration loading paths reject invalid inputs with structured error notifications during automated test verification.

## Enforcement

- Verified by: Automated test suites executing invalid payload scenarios against schema definitions.
- Verified by: Continuous integration static checks verifying type compatibility across schema parsers.
- Verified by: Peer code reviews verifying that new external ingestion paths declare explicit schemas.
- Violation handling: Pull requests introducing unvalidated deserialization on external inputs must be blocked.
- Violation handling: Schema parsing failures at runtime must log actionable diagnostic messages and abort initialization.
- Exception process: Submit an architecture review request detailing the external boundary constraints and compensating controls.
- Exception process: Obtain sign-off from the technical lead and document the approved exception with an in-code justification.