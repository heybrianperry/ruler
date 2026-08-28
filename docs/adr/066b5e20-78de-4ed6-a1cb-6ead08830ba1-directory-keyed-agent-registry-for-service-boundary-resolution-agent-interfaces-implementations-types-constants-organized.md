# Directory-Keyed Agent Registry for Service Boundary Resolution: Agent Interfaces Implementations Types Constants Organized

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The system requires configuration-driven agent selection where different directory contexts may have distinct agent collections
- Agent resolution must support fallback behavior when a configuration directory has no explicitly registered agent collection
- The library layer (`src/lib.ts`) serves as the central coordination point for agent lifecycle and configuration management
- Module boundaries separate agent interfaces, implementations, types, and constants to enforce architectural layering

## Problem Statement

The system needs a mechanism to resolve agent collections based on configuration directory context while supporting fallback semantics when no agents are registered for a given directory. This resolution must occur at a service boundary that coordinates between configuration management and agent selection without tightly coupling directory structure to agent implementation.

## Decision

1. SHOULD: Agent interfaces, implementations, types, and constants SHOULD be organized into separate modules to enforce boundary separation

## Policy Block

- SHOULD Agent interfaces, implementations, types, and constants SHOULD be organized into separate modules to enforce boundary separation

In scope:
- Agent resolution logic that depends on configuration directory context
- Components in the library layer that coordinate agent selection and configuration
- Code paths that require fallback behavior when agent collections are not explicitly registered

Out of scope:
- Direct agent instantiation that does not require directory-scoped resolution
- Static agent configurations that do not vary by directory context
- Agent implementations themselves (which are consumers of this boundary, not implementers)

## Rationale

- The Map-based registry pattern provides O(1) lookup performance for directory-to-agent-collection resolution while maintaining clear ownership of the mapping
- Nullish coalescing enables graceful degradation when configuration directories have no registered agents, supporting incremental configuration adoption
- Centralizing the registry in the library layer establishes a single source of truth for agent resolution, preventing inconsistent agent selection across the codebase
- Module boundary separation (interfaces, implementations, types, constants) enforces architectural layering and reduces coupling between configuration management and agent implementation details

## Consequences

Positive:
- Clear service boundary for agent resolution that decouples configuration directory structure from agent implementation
- Fallback semantics enable robust handling of incomplete or evolving configuration scenarios
- Centralized registry simplifies debugging and auditing of agent selection decisions
- Module separation enforces architectural boundaries and improves testability

Negative:
- Single-file observation limits confidence in cross-cutting adoption of this pattern
- Map-based registry introduces runtime state that must be correctly initialized before agent resolution
- Fallback behavior via nullish coalescing may mask configuration errors if not properly logged
- Directory-keyed resolution couples agent selection to directory structure, which may limit flexibility if directory organization changes

## Alternatives

- Direct agent instantiation at configuration load time without a central registry (rejected)
  Rejected because: Would scatter agent selection logic across configuration loading code, making it difficult to audit which agents are active for a given directory context
  When valid: In systems with static agent configurations that never vary by directory context
- Function-based resolution that computes agent collections on-demand from directory properties (rejected)
  Rejected because: Would require re-computation on every access and complicate caching strategies; the Map registry provides explicit control over when agent collections are determined
  When valid: When agent selection logic is complex enough to require dynamic computation based on runtime directory state beyond simple key lookup
- Dependency injection container that manages agent lifecycle and resolution (deferred)
  When valid: If agent resolution requirements expand to include lifecycle management, scoping, or complex dependency graphs beyond directory-based lookup

## Risks

- Single-file observation may not represent a project-wide architectural pattern; the registry may be localized to one module
  Mitigation: Conduct broader codebase analysis to identify other agent resolution patterns; if inconsistent patterns exist, consolidate or document the boundary between approaches
  Owner: engineering team
- Fallback behavior via nullish coalescing may silently return undefined, masking configuration errors
  Mitigation: Add logging or assertions when fallback paths are taken; consider explicit error handling for critical resolution paths
  Owner: engineering team
- Registry initialization order dependencies may cause resolution failures if agents are queried before registration completes
  Mitigation: Document initialization order requirements; consider lazy initialization or validation checks that ensure registry is populated before first access
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
- Ensure the registry Map is initialized before any configuration loading logic attempts to register agent collections; consider using a module-level singleton or initialization function
- When implementing fallback behavior, add logging or telemetry to track when nullish coalescing returns undefined, enabling detection of missing configuration registrations
- Maintain clear separation between the registry (service boundary) and agent implementations; agents should not directly manipulate the registry

## Continuation Context


Verify commands:
- Locate the library coordination module in the source tree and verify it contains a Map-based registry for directory-to-agent-collection associations
- Search the codebase for registry access patterns and confirm they use nullish coalescing for fallback behavior
- Inspect module import structure to verify separation between agent interfaces, implementations, types, and constants

Accept when:
- The registry Map is present in the library layer and uses configuration directory identifiers as keys
- Agent collection retrieval employs nullish coalescing operators for fallback semantics
- Module boundaries separate agent interfaces, implementations, types, and constants into distinct import paths

## Enforcement

- Verified by: Code review verification that agent resolution uses the central registry rather than direct instantiation
- Verified by: Static analysis to detect agent collection access patterns that bypass the registry
- Verified by: Architecture review to ensure module boundaries between interfaces, implementations, and types are maintained
- Violation handling: Code review feedback requiring refactoring to use the registry pattern
- Violation handling: Architecture review escalation if violations indicate a need to revise the service boundary design
- Violation handling: Documentation updates if legitimate exceptions are discovered that require alternative resolution patterns
- Exception process: Document the specific use case that requires bypassing the registry pattern
- Exception process: Obtain architecture review approval if the exception introduces an alternative service boundary pattern
- Exception process: Update this ADR with policy exceptions if the use case represents a valid scope exclusion