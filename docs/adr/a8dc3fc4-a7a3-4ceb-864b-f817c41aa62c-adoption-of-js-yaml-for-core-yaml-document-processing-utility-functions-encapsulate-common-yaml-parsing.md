# Adoption of js-yaml for Core YAML Document Processing: Utility Functions Encapsulate Common Yaml Parsing

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Core runtime services and agent processing modules require structured document and configuration parsing capabilities.
- Standardizing on a single YAML processing library prevents divergent parsing semantics and redundant dependency footprints across core modules.
- Input validation and safe loading mechanisms are required to safely ingest configuration contents into internal execution structures.

## Problem Statement

Core processing components require a consistent and secure method to parse and deserialize YAML-formatted configurations and documents without introducing disparate parser dependencies or inconsistent parsing behaviors across internal modules.

## Decision

1. MAY: Utility functions MAY encapsulate common js-yaml parsing options to provide uniform error handling across core modules.

## Policy Block

- MAY Utility functions MAY encapsulate common js-yaml parsing options to provide uniform error handling across core modules.

In scope:
- Core processing and utility modules performing YAML parsing, serialization, or configuration loading.

Out of scope:
- Modules handling non-YAML structured document formats or external communication protocols.

## Rationale

- Adopting js-yaml across core processing services ensures uniform parsing semantics and schema translation for YAML documents.
- Restricting YAML processing to a single library minimizes the dependency surface and simplifies version management across the codebase.
- Consolidating parsing routines facilitates centralized security controls and safe deserialization practices.

## Consequences

Positive:
- Standardized parsing behavior across all core subagent processors and utility modules.
- Reduced bundle and dependency footprint by eliminating divergent YAML parser libraries.
- Simplified auditability and maintenance for input validation and document deserialization security.

Negative:
- Coupling of core configuration loading workflows to the specific API semantics of the chosen library.
- Ongoing maintenance obligation to audit and track dependency security advisories for external deserialization libraries.

## Alternatives

- Native or custom bespoke YAML parsing implementation (rejected)
  Rejected because: Custom parsers introduce substantial maintenance overhead and a heightened risk of parsing edge cases or security vulnerabilities compared to established libraries.
  When valid: Valid only in environments with zero-external-dependency constraints or extremely restricted runtime footprints.
- Adoption of alternate multi-format parsing engines (rejected)
  Rejected because: Introducing broader multi-format parser engines adds unnecessary complexity when dedicated libraries meet format-specific requirements directly.
  When valid: Valid when heterogeneous document formats must be parsed interchangeably through a unified polymorphic interface.

## Risks

- Deserialization vulnerabilities from untrusted or malformed YAML inputs.
  Mitigation: Enforce safe parsing modes and comprehensive input validation schemas prior to consuming parsed payloads.
  Owner: Core Engineering Team
- Breaking API changes or deprecation in upstream library releases.
  Mitigation: Strict lock-file version grounding and encapsulation of library calls within utility modules.
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
- Isolate document parsing calls within utility modules to decouple core business logic from direct library API details.
- Apply structural validation immediately after document deserialization to ensure parsed objects conform to expected internal data contracts.

## Continuation Context


Verify commands:
- Discover the repository build script from the project manifest and execute the primary build target.
- Discover the automated test suite runner from the project manifest and execute unit and integration test suites covering document processing modules.
- Discover the static analysis and linting script from the project manifest and run source code validation across core modules.

Accept when:
- All core modules parsing YAML documents invoke js-yaml consistently without secondary parser imports.
- Repository verification and automated test suites confirm successful parsing and validation across document processing workflows.
- Dependency lock artifacts confirm resolved library versions match project specifications.

## Enforcement

- Verified by: Automated static analysis and linting rules that detect unapproved YAML parser imports.
- Verified by: Peer code reviews verifying safe parsing options and schema validation on deserialized payloads.
- Verified by: Continuous integration test passes validating document processing modules.
- Violation handling: Pull requests introducing unauthorized parser dependencies or unsafe loading calls are blocked at review.
- Violation handling: Violations identified during static analysis require refactoring to use the approved library and validation wrappers.
- Exception process: Teams requiring alternative parser behaviors must submit an architectural exception request detailing technical constraints.
- Exception process: Exceptions require approval from the lead architectural maintainers and documented rationale in repository review records.