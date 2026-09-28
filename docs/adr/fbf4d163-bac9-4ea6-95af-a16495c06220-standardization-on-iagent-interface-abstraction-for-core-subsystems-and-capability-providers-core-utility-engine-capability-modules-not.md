# Standardization on IAgent Interface Abstraction for Core Subsystems and Capability Providers: Core Utility Engine Capability Modules Not

Status: proposed
Date: 2025-05-18
Deciders: Detection Pipeline (automated)

## Context

- Core orchestration components including configuration parsing, revert execution, agent selection, and protocol capability exposure require interaction with agent entities.
- Direct dependencies on concrete agent implementations create tight coupling and inhibit modular agent extension or dynamic agent selection.
- The codebase establishes a shared contract boundary via the IAgent abstraction across distinct functional subsystems to enforce dependency inversion.

## Problem Statement

Direct coupling between core orchestration logic and concrete agent implementations creates brittle dependency trees, hampers unit testability, and prevents polymorphic agent selection across runtime subsystems.

## Decision

1. MUST_NOT: Core utility, engine, and capability modules MUST NOT introduce direct import dependencies on concrete agent implementation classes.

## Policy Block

- MUST_NOT Core utility, engine, and capability modules MUST NOT introduce direct import dependencies on concrete agent implementation classes.

In scope:
- All core orchestration subsystems, revert engines, selection modules, and protocol capability providers that interact with agent instances.

Out of scope:
- Agent factory registries responsible for instantiating concrete agent instances.
- Isolated test mocks and harness fixtures that construct synthetic test objects.

Exceptions:
- EXC-20-001: Factory instantiation modules specifically designated for constructing concrete agent instances.

## Rationale

- Cross-cutting imports across configuration, state reversion, selection, and protocol capabilities confirm the IAgent contract as the primary architectural boundary for agent operations.
- Applying dependency inversion guarantees that core operational engines remain agnostic to internal agent execution mechanics.
- Standardizing on the IAgent interface contract enables reliable capability negotiation and predictable behavior across varied operational contexts.

## Consequences

Positive:
- Enforces strict architectural decoupling between core orchestration subsystems and concrete agent execution logic.
- Facilitates straightforward mocking and stubbing of agent behaviors in automated unit tests.
- Enables polymorphic agent selection and capability discovery across multiple engine subsystems.

Negative:
- Requires maintaining interface synchronization whenever underlying agent capabilities or operational requirements evolve.
- Introduces an additional layer of abstraction requiring developers to navigate contract definitions rather than direct implementations.

## Alternatives

- Direct coupling to concrete agent implementation classes in core workflows (rejected)
  Rejected because: Tight coupling impedes substitution of diverse agent models, creates circular dependencies, and complicates isolated unit testing.
  When valid: Prototyping single-agent scripts where polymorphism and runtime agent selection are unnecessary.
- Dynamic duck typing and untyped dictionary dispatch for agent invocation (rejected)
  Rejected because: Eliminates compile-time interface enforcement, increases defect rates in capability negotiation, and obscures agent requirements.
  When valid: Extremely dynamic environments lacking compile-time typing infrastructure.

## Risks

- Interface bloat in the IAgent contract leading to leaky abstractions across dissimilar agent specializations.
  Mitigation: Establish strict contract evolution review processes and maintain cohesive, minimal interface surface areas.
  Owner: engineering team
- Runtime behavior drift between different concrete implementations fulfilling the same IAgent contract.
  Mitigation: Ensure test suites run verification against shared contract compliance test suites.
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
- Implementations interacting with agent instances must depend strictly on the properties and lifecycle methods declared on the IAgent interface contract.
- Agent instantiation and lifecycle management must be decoupled from consumption sites through dedicated factory or selection mechanisms.

## Continuation Context


Verify commands:
- Discover the repository type check script from the configuration manifest and execute it to verify contract compliance.
- Execute the repository linting and import rule verification script to ensure no forbidden concrete agent dependencies exist in core modules.
- Run the repository automated test suite to validate agent contract execution across all dependent subsystems.

Accept when:
- Static analysis verifies that all agent consumers import and depend exclusively on the IAgent contract abstraction without referencing concrete agent classes.
- Type checking and test execution scripts defined in the repository pass with zero errors across all core and protocol capability modules.

## Enforcement

- Verified by: Automated continuous integration checks executing repository type verification and architectural import linting scripts.
- Verified by: Mandatory peer code review verifying that new core subsystem modules consume agent abstractions exclusively through the IAgent contract.
- Violation handling: Pull requests introducing direct dependencies from core subsystems to concrete agent implementations are blocked by automated verification scripts.
- Violation handling: Violations identified during code review must be refactored to consume the IAgent interface before approval.
- Exception process: Deviations from the IAgent contract boundary require an architectural review exception approved by lead maintainers prior to merging.
- Exception process: Temporary exceptions must specify a migration milestone and track follow-up remediation in the project issue tracker.