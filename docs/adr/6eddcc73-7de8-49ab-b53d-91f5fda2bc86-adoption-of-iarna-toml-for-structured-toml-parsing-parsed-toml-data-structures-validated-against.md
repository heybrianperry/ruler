# Adoption of @iarna/toml for Structured TOML Parsing: Parsed Toml Data Structures Validated Against

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Multiple subsystems across core configuration management, agent execution, and protocol synchronization require deserializing TOML content into memory.
- Standardizing on a single dedicated TOML parser prevents behavioral discrepancies, syntax divergence, and disparate error handling across modules.
- Parsing operations must be paired with schema validation downstream to ensure structural correctness of parsed configuration objects.

## Problem Statement

Heterogeneous components across the application independently ingest configuration and metadata serialized in TOML format. Without a standardized parsing library and protocol, parsing discrepancies, varying specification compliance, and uncoordinated dependency usage introduce integration defects and maintenance overhead across configuration, agent, and server propagation workflows.

## Decision

1. SHOULD: Parsed TOML data structures SHOULD be validated against domain schemas immediately following successful parsing before being propagated into runtime state.

## Policy Block

- SHOULD Parsed TOML data structures SHOULD be validated against domain schemas immediately following successful parsing before being propagated into runtime state.

In scope:
- Modules responsible for loading, parsing, and converting TOML configuration or metadata into internal data structures.
- Agent definitions, configuration loaders, and protocol synchronization modules that ingest TOML input.

Out of scope:
- Components handling non-TOML serialization formats such as JSON or YAML.
- Components that consume already-parsed and validated configuration objects in memory.

## Rationale

- The @iarna/toml library provides consistent TOML specification compliance across agent configuration, server propagation, and core config loading boundaries.
- Standardizing on parseTOML across modules eliminates divergent parser edge-case behaviors and reduces redundant third-party dependencies across the codebase.
- Pairing @iarna/toml deserialization with schema validation establishes a uniform and predictable data ingestion boundary.

## Consequences

Positive:
- Enforces unified TOML parsing behavior and consistent specification compliance across all configuration ingestion boundaries.
- Reduces dependency footprint and prevents duplicate serialization libraries within the codebase.
- Establishes a clean separation between syntax-level parsing and downstream domain schema validation.

Negative:
- Creates a codebase-wide coupling to @iarna/toml implementation characteristics and error semantics.
- Upstream updates or breaking changes in the TOML parser require coordinated verification across multiple dependent subsystems.

## Alternatives

- Ad-hoc multi-library TOML parsing (rejected)
  Rejected because: Permitting disparate subsystems to select arbitrary TOML parsers introduces divergent specification compliance, inconsistent error formats, and dependency bloat.
  When valid: Valid only in completely decoupled standalone services that share no internal architecture or standards.
- Custom in-house TOML parsing implementation (rejected)
  Rejected because: Building and maintaining a bespoke TOML parser introduces unnecessary maintenance overhead and risks non-compliance with the full TOML specification.
  When valid: Valid only in highly constrained runtime environments where third-party dependencies are prohibited.

## Risks

- Uncaught syntax errors during TOML parsing leading to unhandled runtime exceptions during configuration load.
  Mitigation: Wrap parseTOML calls in structured error-handling routines that capture parsing failures and emit descriptive diagnostics.
  Owner: engineering team
- Incompatibilities between parsed raw object structures and downstream schema expectations.
  Mitigation: Enforce strict schema validation immediately following parser invocation before exposing configuration values.
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
- Ensure raw string inputs passed to parseTOML originate from validated filesystem reads or trusted memory buffers.
- Isolate parser invocation logic within dedicated loader routines to prevent direct parser dependency leakage into business logic.

## Continuation Context


Verify commands:
- # Discover the project manifest and execute the primary automated test suite to verify configuration parsing routines
- # Discover and run project static analysis scripts to verify dependency imports conform to the designated parser

Accept when:
- All automated test suites covering TOML parsing across configuration, agent, and protocol synchronization pass cleanly.
- Static analysis confirms no alternate TOML parsing libraries are imported or invoked across the codebase.

## Enforcement

- Verified by: Automated continuous integration test suites verifying configuration parsing behavior.
- Verified by: Static analysis checks verifying only @iarna/toml is imported for TOML deserialization.
- Verified by: Peer code reviews on all modifications to configuration ingestion pathways.
- Violation handling: Pull requests introducing unauthorized parsing libraries or unvalidated parseTOML invocations are blocked until aligned with standards.
- Exception process: Exceptions require documented architectural justification and formal review approval from the engineering leads.