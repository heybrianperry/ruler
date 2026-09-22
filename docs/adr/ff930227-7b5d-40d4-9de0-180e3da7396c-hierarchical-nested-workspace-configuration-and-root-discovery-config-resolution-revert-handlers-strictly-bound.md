# Hierarchical nested workspace configuration and root discovery: Config Resolution Revert Handlers Strictly Bound

Status: proposed
Date: 2025-02-18
Deciders: AI (signal conversion)

## Context

- In monorepo setups and subdirectories, previous implementations failed to resolve configuration roots correctly or applied top-level configurations unconditionally.
- This behavior led to agent settings leaking across nested repository boundaries.
- UnifiedConfigLoader, ConfigLoader, and CLI entry points were reworked to discover repository roots from subdirectories, evaluate agent options hierarchically, and scope apply/revert lifecycles to nested project configurations.

## Problem Statement

Should nested configuration discovery automatically merge ancestor ruler configs or enforce strict local isolation to prevent agent settings from leaking across nested repository boundaries?

## Decision

1. MUST: Config resolution and revert handlers MUST strictly bound agent scoping to the nearest discovered workspace root unless explicit inheritance is defined.

## Policy Block

- MUST Config resolution and revert handlers MUST strictly bound agent scoping to the nearest discovered workspace root unless explicit inheritance is defined.

## Rationale

- Strictly bounding agent scoping to the nearest discovered workspace root prevents top-level or ancestor configurations from leaking across nested project boundaries in monorepo environments.
- Ensures lifecycle operations, such as apply and revert handlers, operate solely within the context of the identified workspace root.

## Consequences

Positive:
- Prevents agent configuration leakage across nested repository boundaries.
- Ensures deterministic configuration resolution and lifecycle scoping for subdirectories and monorepo projects.

Negative:
- Workspaces requiring ancestor configuration sharing must explicitly configure inheritance.

## Alternatives

- Automatically merge ancestor ruler configs across nested boundaries (rejected)
  Rejected because: Applying top-level configs unconditionally leaks agent settings across nested repository boundaries in monorepos.

## Risks

- Sub-projects expecting implicit inheritance from root configurations may experience missing settings.
  Mitigation: Provide a mechanism for explicit inheritance when scoping rules across roots.
  Owner: UnifiedConfigLoader and ConfigLoader maintainers

## Implementation Notes

- Update UnifiedConfigLoader, ConfigLoader, and CLI entry points (handlers.ts) to discover workspace roots from subdirectories.
- Scope apply and revert lifecycles (revert.ts, handlers.ts) strictly to the nearest discovered root.
- Verify root detection and agent isolation through nested config CLI and root agent detection integration tests.

## References

- src/core/UnifiedConfigLoader.ts
- src/core/ConfigLoader.ts
- src/cli/handlers.ts
- src/revert.ts
- src/lib.ts
- src/core/FileSystemUtils.ts
- tests/integration/nested-config-cli.test.ts
- tests/integration/root-agents-detection.test.ts
- PR #758
- PR #741
- PR #738
- PR #731
- PR #718
- Commit: 2d19e1eeaf33344781b836f0d3aed78ccc0fdcf7
- Commit: a9af380dfb13d8e768ee938dfe9b034736b2ed49
- Commit: f19f2f00112ac53a4660b18b5261583b8f97514c
- Commit: 5f4e772fa6519c80cb57949de2d866123063b006
- Commit: f16bebcb9d9187c44d886acc70f08f5ec7870e6d
- Commit: 5f3c11d27af7868e07dd13742ff0fb407f37046d
- Commit: 54a4f546341f8683a22feb5cb603aad2ee22e182
- Commit: 12611ef580453e8a3e64a8751ec48c8c4dee7080
- Head commit: 0fa2caeef4efaef4c5f8ffbedee8feca4b0b00e2