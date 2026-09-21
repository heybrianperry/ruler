# IAgent Service Interface Boundary and Directory-Scoped Resolution: Service Resolution Lookups Fall Back Root

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- The codebase coordinates multi-directory operations that configure and invoke specialized agent services.
- Agent services must observe configuration settings specific to their target directory without cross-contaminating state with adjacent directories.
- Both forward execution and reversal workflows require consistent lookup and caching of active agent services keyed by directory.

## Problem Statement

Managing services that operate across multiple configuration directories risks state leakage and conflicting behaviors when services are instantiated globally. Without formal boundary definitions and scoped resolution mechanisms, components cannot reliably isolate configuration settings per directory or fallback to root configurations during multi-directory processing and reversion tasks.

## Decision

1. SHOULD: Service resolution lookups SHOULD fall back to root configuration directory entries when sub-directory specific agent configurations are not present.

## Policy Block

- SHOULD Service resolution lookups SHOULD fall back to root configuration directory entries when sub-directory specific agent configurations are not present.

In scope:
- Invocation and execution of agent services across directory boundaries
- Configuration loading and agent resolution workflows
- Execution and reversion workflows managing directory-scoped agent lifecycles

Out of scope:
- Stateless utility routines that do not depend on configuration directories or agent contracts
- Core path and filesystem operations that do not manage agent lifecycles

Exceptions:
- EXC-23-001: A process requires a temporary global agent mock exclusively within isolated automated test suites

## Rationale

- Binding IAgent service resolution to specific directory paths guarantees that configuration scopes remain strictly segregated across distinct directory boundaries.
- Caching agent instances in directory-keyed maps avoids redundant service resolution overhead while preserving multi-directory isolation.
- Providing fallback to root directory configuration preserves system consistency when sub-directories omit specialized agent definitions.

## Consequences

Positive:
- Strict service encapsulation prevents configuration bleed across independent directory trees.
- Directory-keyed caching optimizes performance by avoiding repeated resolution of agent services.
- Consistent boundary interfaces across execution and reversion workflows reduce operational discrepancies.

Negative:
- Memory overhead increases when maintaining separate agent service instances across numerous directories.
- Resolution logic requires explicit directory hierarchy handling to manage fallback lookups.

## Alternatives

- Global Singleton Agent Service Registry (rejected)
  Rejected because: Global service registries do not support differing per-directory configurations and introduce mutable shared state risks across directory trees.
  When valid: Valid only in single-tenant environments with uniform, static global configuration.
- Ad-hoc Per-Invocation Agent Instantiation Without Caching (rejected)
  Rejected because: Re-resolving and instantiating agent services on every invocation causes excessive filesystem and configuration parsing overhead.
  When valid: Valid in lightweight CLI utilities executed strictly once per process lifecycle.

## Risks

- Cache staleness if underlying directory configurations are updated dynamically during runtime
  Mitigation: Evict or refresh entries in the directory cache whenever configuration reload triggers occur
  Owner: Core Architecture Team
- Excessive memory consumption in repositories containing hundreds of distinct configuration sub-directories
  Mitigation: Bound cache capacity using eviction policies or prune inactive directory entries post-execution
  Owner: Core Architecture Team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Implement service resolution using a mapping structure that associates directory paths with their respective resolved IAgent collection.
- Ensure lookup functions check the specific directory entry first and query root configuration entries as a fallback when the key is absent.

## Continuation Context


Verify commands:
- Discover the project test runner from repository manifests and run all unit and integration test suites.
- Discover the project type checking and linting tools from repository configuration and verify type compliance for service boundary definitions.

Accept when:
- All test suites verifying multi-directory agent execution and reversion pass without failure.
- Static type analysis verifies that all agent instances implement the IAgent contract.
- Directory-scoped caching correctly separates agent configurations across distinct directory paths.

## Enforcement

- Verified by: Automated continuous integration test execution
- Verified by: Peer code review for pull requests modifying service resolution or agent definitions
- Verified by: Static type verification ensuring adherence to service interfaces
- Violation handling: Pull requests bypassing the service boundary or instantiating global state will be rejected.
- Violation handling: Automated build failures on type check or test regression failures.
- Exception process: Exceptions must be submitted via architectural review with documented justification and test isolation proof.