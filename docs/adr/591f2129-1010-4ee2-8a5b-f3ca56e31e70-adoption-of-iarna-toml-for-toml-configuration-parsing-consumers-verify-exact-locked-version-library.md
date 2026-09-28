# Adoption of @iarna/toml for TOML Configuration Parsing: Consumers Verify Exact Locked Version Library

Status: proposed
Date: 2025-02-18
Deciders: Detection Pipeline (automated)

## Context

- Multiple runtime modules including configuration loaders, agent definition processors, and rule application engines consume structured configuration defined in TOML format.
- Ad-hoc or disparate parsing strategies across modules risk divergent syntax interpretations, inconsistent error propagation, and formatting regressions.
- Standardizing on a single TOML parsing implementation ensures unified deserialization behavior across all core subsystems.

## Problem Statement

Modules responsible for configuration loading, agent processing, and rule application require dependable TOML parsing. In the absence of a standardized TOML parser, components risk inconsistent syntax handling, divergent error semantics, and unnecessary reinvention of format parsing algorithms.

## Decision

1. MUST: Consumers MUST verify the exact locked version of the library from the repository dependency lock artifact prior to implementation.

## Policy Block

- MUST Consumers MUST verify the exact locked version of the library from the repository dependency lock artifact prior to implementation.

In scope:
- All subsystems, loader modules, agent controllers, and engine components parsing TOML configuration files, manifests, or text blocks.

Out of scope:
- Modules processing non-TOML serialized payloads such as JSON, YAML, or plain text streams governed by separate format handlers.

## Rationale

- The library @iarna/toml provides compliant, high-performance TOML parsing across configuration loading and execution engines.
- Adopting a single parsing library across the codebase prevents syntax handling divergences between configuration loaders and runtime agents.
- Relying on an established library eliminates the risk of bespoke parsing flaws and ensures consistent validation of incoming structured data.

## Consequences

Positive:
- Standardizes TOML configuration and manifest parsing logic across loaders, agents, and execution engines.
- Delegates TOML specification conformance to a dedicated, battle-tested parsing implementation.
- Provides consistent error handling and syntax validation across disparate entry points.

Negative:
- Adds an external runtime dependency that must be monitored for maintenance, security notices, and upgrades.
- Synchronous parsing operations may block the execution thread if extraordinarily large configuration payloads are parsed.

## Alternatives

- Custom Regular Expression and String Delimiter Parser (rejected)
  Rejected because: Fails to guarantee full TOML specification compliance, struggles with nested tables and array types, and imposes long-term maintenance costs.
  When valid: Acceptable only in zero-dependency bootstrap routines where only trivial key-value pairs are extracted.
- Alternative TOML Parser Libraries (rejected)
  Rejected because: Introducing alternative TOML libraries creates divergent parsing behavior and increases the project dependency footprint when @iarna/toml is already established.
  When valid: Valid only when asynchronous stream-based parsing or distinct runtime engine support is required.

## Risks

- Synchronous parsing of excessively large or malformed TOML inputs could block the execution thread.
  Mitigation: Enforce input size constraints and schema validation before parsing large or untrusted inputs.
  Owner: Core Engineering Team
- Upstream dependency stagnation or breaking changes between major releases.
  Mitigation: Enforce lock-file version resolution policies and subscribe to dependency vulnerability disclosures.
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
- Encapsulate TOML deserialization calls within centralized loader utilities to normalize exceptions and decouple downstream consumers from parser internals.
- Catch syntax and parsing errors raised during deserialization and wrap them in domain-specific configuration errors with actionable diagnostics.

## Continuation Context


Verify commands:
- Discover and execute the project test runner to verify that all configuration and manifest parsing unit tests pass.
- Discover and execute the project static analysis and linting tools to ensure imports comply with approved dependency boundaries.

Accept when:
- All test suites exercising configuration parsing, agent definitions, and rule execution pass successfully.
- Static analysis verifies that TOML parsing imports adhere to the approved core library without unauthorized alternatives.

## Enforcement

- Verified by: Automated continuous integration test suites covering all configuration loading and agent processing workflows.
- Verified by: Static analysis rules verifying dependency import boundaries across the codebase.
- Verified by: Architecture and peer code reviews for any new configuration parsing routines.
- Violation handling: Code reviews and pull request gates will block merging changes that introduce unapproved parsers or custom parsing logic.
- Violation handling: Violations discovered in codebase scans will generate refactoring tasks to replace rogue implementations with the standard library.
- Exception process: Exceptions require an architecture review demonstrating explicit runtime incompatibility or specialized streaming requirements unmet by the adopted library.
- Exception process: Approved exceptions must document boundary isolation and migration plans to prevent pattern leakage.