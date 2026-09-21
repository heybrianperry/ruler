# js-yaml Adoption for Configuration Serialization and Parsing: Modules Parsing External Untrusted Yaml Input

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Core subagent processing modules require structured configuration specifications to define agent behaviors, instructions, and execution boundaries.
- Multiple core components, including subagent utilities and processor pipelines, independently require deserialization and serialization of configuration content.
- Unifying configuration parsing around an established library prevents divergent parsing semantics and inconsistent error reporting across core services.

## Problem Statement

Without a unified serialization and deserialization library for configuration documents, components risk adopting inconsistent parsing semantics, divergent error responses, and potential security vulnerabilities when ingesting untrusted or malformed configuration definitions.

## Decision

1. MUST: Modules parsing external or untrusted YAML input MUST use safe parsing methods and validate parsed output structures against expected schema types prior to downstream ingestion.

## Policy Block

- MUST Modules parsing external or untrusted YAML input MUST use safe parsing methods and validate parsed output structures against expected schema types prior to downstream ingestion.

In scope:
- Core processing modules and utility functions responsible for serializing or deserializing structured YAML configuration documents.

Out of scope:
- Modules processing unstructured text streams or non-YAML structured document formats.

## Rationale

- Standardizing on js-yaml across core utility and processing modules establishes a uniform document ingestion pipeline for structured agent configurations.
- Consolidating around a single established parser reduces redundant dependency overhead and simplifies schema validation enforcement.
- The library provides reliable parsing routines that support standard document structures while allowing strict schema isolation.

## Consequences

Positive:
- Uniform document parsing and error semantics across all core subagent processing modules.
- Centralized dependency usage eliminating redundant or conflicting document parsing implementations.
- Improved security posture by standardizing safe document deserialization practices and schema verification.

Negative:
- Core processing components become coupled to the specific interface and error-handling behavior of the adopted library.
- Payloads of arbitrary depth or complexity necessitate dedicated validation logic to avoid performance degradation during parsing.

## Alternatives

- Adopting custom regular expression or string-splitting implementations for configuration document parsing (rejected)
  Rejected because: Custom parsing logic increases maintenance burden, fails to adhere reliably to standard specifications, and introduces parsing defect risks.
  When valid: Valid only in extremely constrained runtime environments where zero external dependency adoption is permitted.
- Standardizing exclusively on alternative data representation formats without supporting structured YAML configuration documents (rejected)
  Rejected because: YAML provides human readability and multiline string representation essential for defining complex agent prompts and behavior parameters.
  When valid: Valid when configuration documents are strictly machine-generated and never authored or maintained by human operators.

## Risks

- Unsafe loading of untrusted document payloads leading to unintended object instantiation or denial of service.
  Mitigation: Enforce safe parsing functions and strictly validate document shapes against expected schema structures before downstream processing.
  Owner: engineering team
- Breaking API changes across major library dependency upgrades affecting core parsing operations.
  Mitigation: Encapsulate library calls within core utility abstractions and verify against dependency lock artifacts before updating.
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
- Configure parsing routines to enforce strict schema adherence and disable arbitrary runtime object instantiation.
- Encapsulate parsing invocations within shared utility abstractions to maintain consistent deserialization options and schema validation across processing components.

## Continuation Context


Verify commands:
- Discover the project test runner configuration from the repository manifest and execute the test suite covering core document parsing and utility operations.
- Discover the static analysis and linting scripts from the project manifest and execute checks to verify uniform dependency imports and absence of unauthorized parsing libraries.

Accept when:
- All automated test suites covering document parsing pass successfully with valid sample configurations and reject malformed document inputs with structured errors.
- Static analysis and import verification checks confirm that only the approved serialization library is imported for YAML processing across core modules.

## Enforcement

- Verified by: Automated unit and integration test suites executed within continuous integration pipelines.
- Verified by: Static analysis and dependency checking checks verifying uniform library import usage across the codebase.
- Verified by: Peer architectural and code review for any pull requests introducing or modifying configuration parsing operations.
- Violation handling: Continuous integration checks fail when unapproved parsing libraries or direct unsafe parsing calls are identified.
- Violation handling: Merge requests containing unapproved parsing dependencies are blocked until aligned with approved standards.
- Exception process: Submit an architectural exception request detailing why the approved library cannot fulfill specific configuration processing requirements.
- Exception process: Obtain formal sign-off from the architectural governance team before introducing any alternative or additional parsing dependencies.