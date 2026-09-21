# AbstractAgent Core Module Adoption for Agent Implementations: Specialized Agent Implementations Override Hook Methods

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Multiple specialized agent integrations require consistent lifecycle management, configuration resolution, and invocation structures across the application.
- Direct standalone agent implementations risk divergence in contract handling, error recovery, and path resolution conventions.
- A centralized base class abstraction establishes a standardized inheritance model for concrete agent implementations while sharing common execution facilities.

## Problem Statement

Without a standardized base abstraction, concrete agent integrations risk duplicating common lifecycle logic, deviating from core interface contracts, and inconsistently resolving runtime paths. This fragmentation increases maintenance overhead and leads to unpredictable behavior across different agent execution environments.

## Decision

1. MAY: Specialized agent implementations MAY override hook methods provided by AbstractAgent to supply provider-specific communication or configuration logic.

## Policy Block

- MAY Specialized agent implementations MAY override hook methods provided by AbstractAgent to supply provider-specific communication or configuration logic.

In scope:
- Implementation and extension of specialized agent modules across the codebase.
- Module architecture and inheritance hierarchy for agent integrations.

Out of scope:
- General utility modules that do not implement agent execution contracts.
- External client software development kit wrappers operating strictly outside the internal agent abstraction layer.

Exceptions:
- EXC-20-001: A lightweight mock or test double is required strictly within automated test fixtures that cannot instantiate the full base class.

## Rationale

- Evidence across multiple concrete agent implementations demonstrates consistent adoption of the AbstractAgent base class to maintain uniform execution patterns.
- Inheriting from a centralized base class reduces implementation redundancy by sharing standard lifecycle behaviors across distinct agent variants.
- Standardizing module organization through a central aggregation export ensures consumers interact with agent implementations through a consistent interface boundary.

## Consequences

Positive:
- Ensures uniform lifecycle, state handling, and execution behavior across all specialized agent implementations.
- Reduces boilerplate and duplicate path and configuration logic when introducing new agent integration types.
- Maintains a clean architectural boundary and predictable public surface exposed through the central agent export index.

Negative:
- Introduces class-level coupling between concrete agent implementations and the internal base class hierarchy.
- Modifications to base class behavior require regression verification across all downstream inheriting agent modules.

## Alternatives

- Independent standalone implementations for each agent module without a shared base class (rejected)
  Rejected because: Increases implementation redundancy and leads to divergent lifecycle handling and inconsistent interface compliance across agent implementations.
  When valid: Valid only in isolated utility environments where agent modules share no common execution contracts or lifecycle phases.
- Pure interface composition using only contract interfaces without an abstract base class (rejected)
  Rejected because: Requires each concrete implementation to replicate identical path resolution, configuration initialization, and boilerplate execution logic.
  When valid: Valid when agent variants execute on fundamentally distinct runtime platforms preventing any shared logic reuse.

## Risks

- Breaking changes in the base class can cascade failures across all inheriting concrete agent modules.
  Mitigation: Enforce strict contract testing and comprehensive integration test suites across all concrete implementations prior to merging base class changes.
  Owner: Architecture Team
- Over-generalization of the base class leading to excessive coupling or bloated abstractions.
  Mitigation: Maintain base class focus strictly on core lifecycle orchestration, delegation, and common path resolution facilities.
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
- When introducing a new agent variant, inherit from AbstractAgent and implement all abstract lifecycle and execution methods defined by the core agent contract.
- Export newly implemented concrete agents through the primary module aggregation file alongside existing agent definitions.

## Continuation Context


Verify commands:
- Discover and execute the repository's test runner to validate that all agent implementations successfully pass lifecycle and contract tests.
- Discover and execute the project's type-checking and linter suites to ensure all agent implementations strictly conform to AbstractAgent contracts.

Accept when:
- All concrete agent modules inherit from AbstractAgent and pass type verification without contract mismatches.
- All agent lifecycle and execution test suites pass without regressions across all agent variants.

## Enforcement

- Verified by: Automated static analysis and type checking during continuous integration pipelines.
- Verified by: Peer code review for all new or modified agent implementations.
- Violation handling: Pull requests containing agent implementations that do not inherit from the base class or fail contract conformance are blocked from merging.
- Exception process: Submit an architectural review request documenting why the specialized agent variant cannot extend AbstractAgent, requiring explicit approval from the architecture team.