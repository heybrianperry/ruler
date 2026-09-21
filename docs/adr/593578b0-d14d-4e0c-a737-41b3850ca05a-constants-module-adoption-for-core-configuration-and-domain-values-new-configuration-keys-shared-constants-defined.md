# Constants Module Adoption for Core Configuration and Domain Values: New Configuration Keys Shared Constants Defined

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Core operational modules across the processing and agent subsystems require access to shared configuration keys, environment variable references, and system operational defaults.
- Hardcoding string literals and configuration constants independently across core processing modules causes configuration drift, increases the risk of subtle naming mismatches, and complicates refactoring.
- Establishing a single centralized constants module provides a unified source of truth for runtime configuration names, environment variables, and default threshold values across processing units.

## Problem Statement

Without a centralized definition for shared system constants, core processing modules risk duplicating literal values for environment variables, configuration keys, and domain parameters. This duplication introduces drift, complicates cross-cutting updates, and creates potential runtime inconsistencies across processors, selection components, and filesystem utilities.

## Decision

1. SHOULD: New configuration keys and shared constants SHOULD be defined with explicit types and exported immutably from the constants module.

## Policy Block

- SHOULD New configuration keys and shared constants SHOULD be defined with explicit types and exported immutably from the constants module.

In scope:
- Core subsystem modules and utilities requiring access to shared system keys, operational defaults, or cross-cutting configuration parameters.
- Components coordinating agent selection, subagent processing, skills processing, and filesystem operations.

Out of scope:
- Module-private variables and calculation constants that have no relevance outside a single isolated translation unit.
- Dynamic user configuration values supplied at runtime from external inputs.

## Rationale

- Centralizing shared constants within a dedicated internal module guarantees that configuration identifiers and environment variable keys remain synchronized across core processors.
- Importing from a common constants module enables compile-time type checking and automated refactoring across all dependent operational modules.
- Decoupling constant definitions from individual processors prevents circular dependencies while establishing clear architectural boundaries.

## Consequences

Positive:
- Provides a single authoritative source of truth for shared configuration identifiers, environment keys, and operational limits.
- Eliminates duplicate literal definitions across core modules, reducing defect risk during configuration updates.
- Simplifies auditing and validation of system defaults and cross-cutting constants.

Negative:
- Requires developers to update a shared central module when adding or modifying cross-cutting configuration keys.
- Introduces a shared dependency that must remain stable to avoid wide-reaching ripple effects across dependent components.

## Alternatives

- In-line literal definitions within individual processing modules (rejected)
  Rejected because: Leads to duplicate literal declarations, inconsistent configuration key naming, and high maintenance overhead when keys change.
  When valid: When a literal is strictly unique to one private function and has no cross-cutting or shared semantics.
- Distributed per-subsystem configuration files (rejected)
  Rejected because: Fragments constant definitions across multiple locations, increasing the likelihood of divergent naming and redundant imports.
  When valid: When subsystems are deployed as completely independent external services with distinct release cycles.

## Risks

- Uncontrolled growth of the central constants module turning it into an unstructured dumping ground for unrelated values.
  Mitigation: Group exported constants into cohesive, logically structured domain groupings and enforce code review standards for new exports.
  Owner: engineering team
- Unintentional breaking changes in shared constants impacting multiple downstream processors simultaneously.
  Mitigation: Use automated regression test suites and type-checking verification scripts to catch downstream breakages before merging changes.
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
- Ensure all constants exported from the central module are defined as read-only or immutable structures to prevent runtime mutation.
- Group constants logically by operational concern to maintain discoverability as the system evolves.

## Continuation Context


Verify commands:
- Discover and run the project static analysis and type verification script to ensure all constant imports resolve correctly.
- Discover and run the project test suite to verify that core processing modules execute with centralized constant definitions without regressions.

Accept when:
- Static analysis and type checking pass with zero errors across all core modules importing from the constants module.
- All unit and integration test suites pass successfully, confirming consistent behavior across core processing workflows.

## Enforcement

- Verified by: Automated static analysis checks in the continuous integration pipeline verifying that core modules do not duplicate shared literal strings.
- Verified by: Peer code review for pull requests modifying or adding shared constants.
- Violation handling: Pull requests introducing duplicate shared literals or bypassing the centralized constants module will be blocked until corrected.
- Exception process: Exceptions for module-private or performance-critical isolated literals must be documented in code comments and approved by the core engineering maintainers.