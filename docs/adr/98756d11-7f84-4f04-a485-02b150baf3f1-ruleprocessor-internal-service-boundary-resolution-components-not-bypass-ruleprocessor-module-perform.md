# RuleProcessor Internal Service Boundary Resolution: Components Not Bypass Ruleprocessor Module Perform

Status: proposed
Date: 2025-02-18
Deciders: Detection Pipeline (automated)

## Context

- Execution engine components require coordination across agent dispatching, file system utilities, and configuration parsing.
- Direct in-memory mapping operations were introduced within execution routines to track agent selections and output path states.
- Service boundaries require explicit encapsulation within internal modules to avoid scattering stateful resolution logic across execution flows.

## Problem Statement

Ad-hoc service definitions and localized in-memory cache lookups create tight coupling between execution coordination and state tracking, risking interface divergence when managing agent lifecycles and file-system artifacts.

## Decision

1. MUST_NOT: Components MUST NOT bypass the RuleProcessor module to perform direct mutable state mutations across service boundaries.

## Policy Block

- MUST_NOT Components MUST NOT bypass the RuleProcessor module to perform direct mutable state mutations across service boundaries.

In scope:
- Internal execution engines coordinating agent configurations and output path verification
- Modules integrating RuleProcessor for boundary management

Out of scope:
- Stateless utility helpers that do not manage agent lifecycles or boundary caches
- Standalone configuration parsers operating outside the execution engine flow

## Rationale

- Encapsulating service definitions within RuleProcessor establishes a single boundary for agent coordination and state lookup.
- Centralizing output path existence checks and agent resolution avoids fragmented cache state across disparate engine functions.
- Constraining service boundaries prevents internal state structures from leaking into higher-level orchestrators.

## Consequences

Positive:
- Defines clear boundaries for agent execution and configuration resolution.
- Reduces redundant disk and path inspection by caching state within the designated boundary.
- Prevents arbitrary state mutations outside of established internal service definitions.

Negative:
- Introduces encapsulation overhead for simple agent lookups.
- Couples lifecycle state to the RuleProcessor boundary lifecycle.

## Alternatives

- Direct unencapsulated in-memory map queries within caller functions (rejected)
  Rejected because: Scatters boundary definitions and lifecycle state tracking across individual calling functions, increasing maintenance complexity.
  When valid: In throwaway prototypes or isolated one-off scripts with no recurring execution state.
- Distributed microservice API boundaries (rejected)
  Rejected because: Introduces disproportionate network and serialization overhead for internal engine execution operations.
  When valid: When execution agents run as independently deployable remote services.

## Risks

- Stale state in cached agent definitions or path existence maps if external files change during execution.
  Mitigation: Enforce explicit invalidation or bounded cache lifetimes across execution cycles.
  Owner: Engineering Team
- Performance bottlenecks if boundary lookups introduce synchronization locks during concurrent processing.
  Mitigation: Keep boundary lookups non-blocking and evaluate lookup frequency under profile testing.
  Owner: Engineering Team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Locate the project dependency manifest and lock artifact to inspect module exports and interface contracts before defining service boundaries.
- Ensure agent selection lookups and path verification routines adhere to the encapsulation layer defined by the RuleProcessor module.

## Continuation Context


Verify commands:
- Discover and run the project test suite to validate service boundary isolation and agent lookup behavior.
- Discover and run the repository linter and static analysis checks to ensure module boundary rules are respected.

Accept when:
- All discovery verification scripts pass without boundary violations or unhandled exceptions.
- Service lookups and path validations execute predictably without leaking unencapsulated map state.

## Enforcement

- Verified by: Automated continuous integration checks executing repository test and lint workflows.
- Verified by: Peer code review verifying adherence to service boundary encapsulation.
- Violation handling: Pull requests exhibiting unencapsulated boundary state lookups must be revised before merge.
- Violation handling: Continuous integration failures due to boundary violations block deployment.
- Exception process: Architectural review and documentation justifying the exemption in the affected module.