# AgentsMdAgent Module Adoption for Agent Implementations: Agent Integration Modules Not Implement Independent

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Multiple specialized agent integration modules require uniform parsing, generation, and lifecycle management for agent configuration state.
- Individual agent implementations previously risked fragmenting configuration handling and markdown formatting logic across divergent integration boundaries.
- The codebase establishes a shared internal module, AgentsMdAgent, alongside the IAgent interface to standardize agent contracts across all supported integration targets.
- Consolidating core agent configuration behaviors into AgentsMdAgent ensures centralized enforcement of serialization and agent document structural conventions.

## Problem Statement

Specialized agent integrations across the codebase require cohesive markdown configuration generation, parsing, and interface conformance. Without standardizing on a shared internal agent module, individual agent implementations duplicate configuration parsing, introduce divergent lifecycle hooks, and increase maintenance overhead across heterogeneous tool integrations.

## Decision

1. MUST_NOT: Agent integration modules MUST NOT implement independent or custom markdown configuration parsers that duplicate functionality established within AgentsMdAgent.

## Policy Block

- MUST_NOT Agent integration modules MUST NOT implement independent or custom markdown configuration parsers that duplicate functionality established within AgentsMdAgent.

In scope:
- Development and maintenance of agent integration modules providing tool-specific or environment-specific agent workflows.
- Shared agent configuration, document generation, and interface definitions within the agent subsystem.

Out of scope:
- General utility modules and platform-agnostic file system helpers that do not implement agent contracts or workflows.
- External consumer applications that interact with the system solely through public entry points without defining new agent types.

Exceptions:
- EXC-20-001: A target agent environment requires an entirely non-document protocol that cannot map to the markdown schema enforced by AgentsMdAgent.

## Rationale

- Grounded static analysis reveals uniform adoption of the internal AgentsMdAgent module across all detected agent implementation modules.
- Standardizing on AgentsMdAgent and the IAgent contract ensures structural consistency, predictable document parsing, and single-point maintenance for agent configuration updates.
- Centralizing agent configuration logic eliminates redundant parsing implementations and reduces defect risk when agent protocol formats evolve.

## Consequences

Positive:
- Provides a unified implementation foundation and contract conformance across all specialized agent integrations.
- Eliminates duplicate parsing and serialization logic across discrete agent targets.
- Simplifies onboarding of new agent integrations through established base classes and contracts.

Negative:
- Creates direct coupling between individual agent implementations and the internal AgentsMdAgent module.
- Changes to AgentsMdAgent contract signatures propagate across all agent implementations, requiring coordinated verification.

## Alternatives

- Autonomous Per-Agent Custom Configuration Implementations (rejected)
  Rejected because: Duplicating configuration generation and markdown formatting across every agent integration leads to contract divergence, inconsistent behavior, and elevated maintenance burden.
  When valid: Valid only in decentralized multi-repository architectures where agent modules share no runtime dependencies.
- Minimal Interface-Only Abstraction Without Shared Base Implementation (rejected)
  Rejected because: Requiring only interface conformance without a shared implementation forces every agent integration to re-implement common markdown parsing and serialization logic.
  When valid: Valid when agent targets share zero structural behavior or configuration schema overlap.

## Risks

- Upstream changes in AgentsMdAgent could inadvertently break specialized agent targets with distinct configuration requirements.
  Mitigation: Maintain comprehensive automated test suites verifying each specialized agent target against the shared module contract.
  Owner: engineering team
- Over-generalization in AgentsMdAgent could constrain tool-specific agent optimizations.
  Mitigation: Allow extension and override hooks within child implementations while keeping base document parsing standardized.
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
- When constructing a new agent integration, extend the AgentsMdAgent base structure and fulfill the IAgent contract to inherit standard configuration lifecycle management.
- Delegate all markdown serialization and schema validation to the inherited AgentsMdAgent facilities rather than introducing local custom parsers.

## Continuation Context


Verify commands:
- Discover the project test runner configuration from the repository manifest and execute the test suite covering agent integration modules.
- Discover the project static analysis and linting configuration from the repository manifest and execute validation across all agent integration components.

Accept when:
- All agent integration modules successfully instantiate and inherit from AgentsMdAgent without contract violations.
- All agent test suites pass cleanly with zero linting and type-checking diagnostics reported across agent implementations.

## Enforcement

- Verified by: Automated continuous integration pipelines validating static types, lint rules, and test suites across all agent modules.
- Verified by: Peer code review verifying that any new or modified agent integration extends AgentsMdAgent and adheres to the IAgent interface.
- Violation handling: Pull requests introducing divergent agent configuration parsers or bypassing AgentsMdAgent are blocked from merge until refactored to use the shared module.
- Exception process: Exceptions require a formal architecture deviation proposal approved by the architecture review board and documented with rationale.