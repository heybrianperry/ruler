# Zod Schema Validation: Deserialization Boundaries Not Pass Unvalidated Objects

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- External configuration and serialized data files enter the application boundary from multiple persistent storage locations and environment paths.
- Unvalidated deserialization allows unexpected or malformed property structures to bypass compile-time type assumptions and cause downstream runtime errors.
- Core configuration parsing incorporates strict schema definitions to enforce explicit runtime validation contracts before runtime access.
- Other serialization boundaries across the codebase predominantly execute direct string deserialization without structural schema verification, creating inconsistent boundary enforcement.

## Problem Statement

Direct deserialization of structured content without schema validation exposes application components to malformed inputs, missing required attributes, and unexpected property structures that degrade runtime reliability. The architecture requires a standardized approach to validate external input structures at boundary crossing points to guarantee type safety and structural integrity.

## Decision

1. MUST_NOT: Deserialization boundaries MUST NOT pass unvalidated objects directly into core domain structures without prior schema validation.

## Policy Block

- MUST_NOT Deserialization boundaries MUST NOT pass unvalidated objects directly into core domain structures without prior schema validation.

In scope:
- Ingestion of external configuration files and structured payloads at application entry points.
- Deserialization of persistent state representations that map to internal domain interfaces.

Out of scope:
- Internal memory-only data structures generated and consumed exclusively within a single trusted module.
- Static assets bundled directly into the distribution without dynamic user or environment configuration.

## Rationale

- IR analysis demonstrates that schema validation using Zod prevents runtime defects by enforcing strict type boundaries on parsed content.
- Allowing unvalidated deserialization across modules introduces disparate failure modes when unexpected properties or invalid data types are encountered.
- Centralizing structural schema validation at boundary layers protects downstream domain logic from handling malformed or unexpected data states.

## Consequences

Positive:
- Guarantees runtime structural validation and type safety for external input payloads.
- Fails fast at input boundaries with actionable structural errors before invalid state reaches core logic.
- Provides clear, single-source-of-truth schemas that mirror static typing expectations.

Negative:
- Adds runtime execution overhead for parsing and schema traversal during startup and file loading.
- Requires ongoing schema maintenance and synchronization when domain data contracts evolve.
- Rejects payloads containing unknown attributes when strict mode is active, requiring explicit schema deprecation paths.

## Alternatives

- Direct unvalidated deserialization with manual type casting (rejected)
  Rejected because: Manual type casting provides compile-time typing without runtime verification, leaving components vulnerable to malformed payloads and undefined property errors.
  When valid: When parsing non-critical internal payloads with trivial structures and negligible failure impact.
- Custom imperative validation functions (rejected)
  Rejected because: Custom imperative validation introduces boilerplate, inconsistent error handling, and high maintenance overhead compared to declarative schemas.
  When valid: When runtime constraints cannot be modeled declaratively through schema definitions.

## Risks

- Strict schema validation may cause application initialization failures if external configuration files contain unrecognized legacy keys.
  Mitigation: Incorporate deprecation fallback schemas and clear migration paths for legacy keys.
  Owner: Core Engineering Team
- Performance degradation during repetitive deserialization of large payload collections.
  Mitigation: Cache validated schema results and restrict schema execution to boundary entry points.
  Owner: Core Engineering Team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Define schemas alongside domain type declarations to keep runtime validation and static types aligned.
- Implement safe parsing methods to collect and log structural validation issues without unhandled runtime crashes.

## Continuation Context


Verify commands:
- Discover and execute the repository verification scripts to run test suites covering boundary parsing and schema validation.
- Discover and execute the repository static analysis and type-checking scripts to verify schema and type alignment.

Accept when:
- All configuration parsing test suites execute successfully and validate both valid and invalid schema inputs.
- Static analysis checks pass with zero type errors across all schema definition modules.

## Enforcement

- Verified by: Automated continuous integration test suites verifying schema validation on test fixtures.
- Verified by: Peer code review checking that newly introduced deserialization points include corresponding schema validation.
- Violation handling: Pull requests introducing unvalidated deserialization at external boundaries will be blocked until schema validation is added.
- Violation handling: Defects arising from unvalidated boundary payloads must be resolved by introducing appropriate schema validation.
- Exception process: Exceptions for performance-critical paths require architectural review and written documentation of compensating controls.