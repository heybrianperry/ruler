# js-yaml Adoption for YAML Processing: Callers Handle Parser Syntax Validation Exceptions

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Core subagent components require loading, parsing, and validating declarative structured configurations stored in YAML format.
- Disparate parsing approaches across processing utilities create inconsistent error behavior and schema interpretation risks.
- Static analysis detects standardized adoption of js-yaml across core processing and utility components alongside input validation routines.

## Problem Statement

Declarative agent configurations and structured definition files must be ingested reliably across core processing utilities without introducing divergent parser behaviors, unvalidated object evaluation, or redundant library dependencies.

## Decision

1. SHOULD: Callers SHOULD handle parser syntax and validation exceptions at the ingestion boundary and transform them into standardized domain errors.

## Policy Block

- SHOULD Callers SHOULD handle parser syntax and validation exceptions at the ingestion boundary and transform them into standardized domain errors.

In scope:
- Core processing modules and utility components requiring YAML document parsing and serialization.

Out of scope:
- Data formats other than YAML and runtime environments where configuration is provided exclusively via pre-parsed memory structures.

## Rationale

- Adopting a single verified parsing library eliminates discrepancies in YAML specification compliance across the codebase.
- Pairing parser invocations with input validation prevents untrusted configuration payloads from inducing runtime exceptions or injection vulnerabilities.
- Centralizing on a recognized module satisfies the observed dependency pattern across core processing components.

## Consequences

Positive:
- Standardizes YAML parsing across utility and core processing modules on a single tested engine.
- Enforces predictable error handling and validation semantics for structured declarative definitions.
- Prevents fragmentation caused by multiple competing YAML parser implementations across components.

Negative:
- Introduces an external runtime dependency requiring continuous security patch tracking and resolution audits.
- Parsing and schema validation introduce execution overhead during initial configuration ingest and processing.

## Alternatives

- Adopting alternative YAML parsing engines or in-house custom parser implementations (rejected)
  Rejected because: Custom parsers increase maintenance overhead and divergence risks, while multiple external engines create conflicting dependency graphs.
  When valid: Valid only if specific target environments exhibit strict binary size limits that preclude standard library inclusion.
- Adopting unvalidated dynamic evaluation of structured configuration payloads (rejected)
  Rejected because: Parsing untrusted structured text without schema or input validation exposes execution paths to unexpected data injection.
  When valid: Valid only in sandboxed test suites with static, non-user-supplied test fixtures.

## Risks

- Parsing untrusted YAML inputs could lead to prototype pollution or resource exhaustion.
  Mitigation: Apply input validation checks and strict schema parsing guards on all deserialized content.
  Owner: engineering team
- Breaking API changes in subsequent dependency updates could disrupt parsing behavior.
  Mitigation: Ground all implementations against the repository resolution artifact per the Discovery Policy.
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
- Encapsulate parsing calls within dedicated utility wrappers to isolate schema validation and error transformation logic.
- Ensure asynchronous file read operations complete before passing raw document content to parser methods.

## Continuation Context


Verify commands:
- Discover the repository test runner from the project manifest and execute the test suite governing configuration parsing.
- Discover the linting and static analysis command from the build configuration and execute validation across core processing modules.

Accept when:
- All configuration parsing modules invoke js-yaml for YAML deserialization and pass verification suites.
- Input validation routines successfully intercept malformed YAML content prior to downstream processing.
- Dependency analysis confirms no secondary YAML parsing libraries exist within core modules.

## Enforcement

- Verified by: Automated dependency linting during integration pipeline runs.
- Verified by: Static code analysis verifying parser imports and validation invocations.
- Verified by: Peer code review on all pull requests modifying configuration ingestion paths.
- Violation handling: Automated build rejection upon detection of unapproved YAML parsing libraries.
- Violation handling: Code review rejection for unvalidated parsing calls on external input.
- Violation handling: Required remediation to refactor non-compliant implementations to standard parser interfaces.
- Exception process: Submit an architectural change request detailing functional or environmental constraints preventing js-yaml adoption.
- Exception process: Obtain formal approval from core module maintainers prior to merging any secondary parser dependencies.
- Exception process: Document approved exceptions with clear boundaries in the module documentation.