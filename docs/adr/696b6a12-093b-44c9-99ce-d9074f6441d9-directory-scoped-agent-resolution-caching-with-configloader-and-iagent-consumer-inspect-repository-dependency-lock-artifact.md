# Directory-Scoped Agent Resolution Caching with ConfigLoader and IAgent: Consumer Inspect Repository Dependency Lock Artifact

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Configuration loaders process hierarchical directory entries to establish execution settings.
- Resolving agent instances repeatedly across root and nested configuration scopes incurs redundant processing.
- An in-memory dictionary keyed by directory path associates resolved IAgent instances with specific configuration contexts.

## Problem Statement

Repeated resolution of agent instances across nested configuration hierarchies creates redundant processing overhead and potential reference discrepancies. A consistent boundary pattern is needed to resolve agents once per configuration directory and expose them deterministically across dependent operations.

## Decision

1. MUST: The consumer MUST inspect the repository dependency lock artifact and confirm that all core runtime modules and interfaces match the recorded resolved versions before modifying service boundary contracts.

## Policy Block

- MUST The consumer MUST inspect the repository dependency lock artifact and confirm that all core runtime modules and interfaces match the recorded resolved versions before modifying service boundary contracts.

In scope:
- Operations that resolve or retrieve IAgent instances across directory-based configuration entries.
- Service boundaries coordinating configuration loader results with agent execution contexts.

Out of scope:
- Global agent registries that operate independently of directory-specific configuration hierarchies.
- Stateless utility routines executing without configuration loader dependencies.

## Rationale

- Mapping resolved IAgent collections by configuration directory avoids duplicate parsing and instantiation across related command operations.
- Reusing resolved instances ensures consistent service definitions between root configurations and directory-specific configurations.
- Local dictionary lookups reduce filesystem traversal overhead during multi-step command lifecycles.

## Consequences

Positive:
- Eliminates redundant agent resolution and filesystem traversal across configuration hierarchies.
- Ensures consistent IAgent instance references across root and local configuration boundaries.
- Provides deterministic agent retrieval through explicit directory-keyed map lookups.

Negative:
- Introduces in-memory state retention that requires lifecycle-bound cleanup.
- Increases runtime heap consumption relative to the number of distinct directory scopes traversed.

## Alternatives

- Resolve IAgent collections on demand during each access without in-memory caching (rejected)
  Rejected because: Causes redundant configuration parsing and repeated agent instantiation across directory hierarchies.
  When valid: When execution environments require strictly stateless operations with negligible resolution overhead.
- Maintain a single global agent registry across all configuration paths (rejected)
  Rejected because: Cannot accommodate differing agent configurations tailored to distinct directory scopes.
  When valid: When agent configuration is universally homogeneous across the entire repository.

## Risks

- In-memory agent mapping can retain stale instances across long-running lifecycles.
  Mitigation: Bind cache lifecycle strictly to the command invocation context so instances are released upon completion.
  Owner: engineering team
- Underlying configuration changes during execution could lead to divergence from disk state.
  Mitigation: Clear or re-instantiate directory mappings whenever configuration files are modified or reloaded.
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
- Directory-scoped agent caches should be scoped strictly to the lifecycle of the invoking command execution to prevent stale instance retention.
- Consumers must interact with agents exclusively through the IAgent interface abstraction.

## Continuation Context


Verify commands:
- Discover and execute the test suite declared in the project repository manifest to validate agent resolution caching.
- Run the repository static analysis and type verification scripts to ensure compliance with the IAgent interface contract.

Accept when:
- All tests verifying directory-keyed agent caching and retrieval pass successfully.
- Static type checking confirms all cached collections strictly satisfy the IAgent contract.

## Enforcement

- Verified by: Automated test suites executed during continuous integration workflows.
- Verified by: Peer code review verifying directory-keyed agent lookup and avoidance of redundant resolution.
- Violation handling: Pull requests introducing redundant resolution or bypassing directory-keyed agent retrieval must be refactored before merging.
- Exception process: Submit an architectural review request outlining why a component requires uncached or alternate agent resolution.