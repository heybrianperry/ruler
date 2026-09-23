# Symlink containment boundary for config resolution and generated outputs: File Mutations Configuration Path Traversals Validate

Status: proposed
Date: 2026-09-23
Deciders: AI (signal conversion)

## Context

- Path resolution and file mutation operations across ConfigLoader, UnifiedConfigLoader, apply-engine, and agent adapters previously performed operations that could traverse symbolic links implicitly.
- Without containment checks at a centralized filesystem utility abstraction, repositories containing malicious or misconfigured symlinks can cause file writes or configuration ingestion to escape the workspace boundary during apply or revert workflows.

## Problem Statement

Direct and uninspected filesystem calls permit symbolic link traversal that can escape workspace boundaries, leading to potential out-of-sandbox file mutations or arbitrary configuration reads.

## Decision

1. MUST: All file mutations and configuration path traversals MUST validate symlink targets against workspace boundaries via FileSystemUtils rather than directly invoking raw fs methods.

## Policy Block

- MUST All file mutations and configuration path traversals MUST validate symlink targets against workspace boundaries via FileSystemUtils rather than directly invoking raw fs methods.

In scope:
- src/core/FileSystemUtils.ts
- src/core/ConfigLoader.ts
- src/core/UnifiedConfigLoader.ts
- src/core/apply-engine.ts
- All agent adapters and configuration resolution modules

Out of scope:
- External build tooling executing entirely outside the application runtime boundary.

Exceptions:
- ex-1: System utilities explicitly operate in a non-sandboxed administrative maintenance mode.

## Rationale

- Centralizing path operations and boundary verification within FileSystemUtils prevents individual loaders and engines from inconsistently handling symlink inspection.
- Enforcing strict boundary validation shields the host environment from unintended escapes during apply and revert operations.

## Consequences

Positive:
- Consistent enforcement of workspace boundary containment across all loaders and execution engines.
- Protection against malicious or accidental filesystem writes and reads outside target workspaces.

Negative:
- Slight performance overhead incurred by resolving and checking symlink target paths on every filesystem interaction.
- Workflows intentionally relying on external symlink targets must be explicitly re-architected or managed.

## Alternatives

- Direct file path operations following symbolic links implicitly without workspace validation (rejected)
  Rejected because: Fails to detect symlink escapes, allowing malicious or misconfigured symlinks to direct mutations and configuration loading outside the repository sandbox.
- Decentralized boundary checks implemented independently inside each loader and adapter (rejected)
  Rejected because: Leads to duplicate logic, inconsistent security guarantees, and high risk of omission in new or existing components.

## Risks

- Legitimate symlinks pointing outside the workspace for shared developer tools or global dependencies may be blocked.
  Mitigation: Ensure workspace boundaries can be explicitly configured or allow scoped exceptions through documented abstraction boundaries.
  Owner: Core Architecture Team

## Implementation Notes

- Path resolution logic in FileSystemUtils must resolve canonical realpaths before comparing against the configured workspace root boundary.
- ConfigLoader, UnifiedConfigLoader, and apply-engine should be audited to replace any remaining native fs calls with FileSystemUtils containment wrappers.

## Continuation Context


Verify commands:
- Discover and execute test suites targeting symlink containment, path traversal safety, and configuration loader boundary checks.
- Discover and execute static analysis checks verifying that core modules do not import raw fs path manipulation methods directly.

Accept when:
- All integration tests validating symlink output safety pass successfully.
- All path resolution and write operations route strictly through FileSystemUtils containment guards.

## Enforcement

- Verified by: Automated integration tests validating symlink containment behavior.
- Verified by: Static analysis and code review checks blocking direct raw filesystem imports in engine and loader modules.
- Violation handling: Pull requests with direct fs method usage in place of FileSystemUtils boundary methods will fail automated linting and code review.
- Violation handling: Runtime execution halts with a containment validation error upon detecting an out-of-bounds symlink.
- Exception process: Submit an architectural change request detailing the justification for external symlink traversal, subject to approval by the architecture review team.

## References

- file:src/core/FileSystemUtils.ts
- file:src/core/ConfigLoader.ts
- file:src/core/UnifiedConfigLoader.ts
- file:src/core/apply-engine.ts
- tests/integration/symlink-output-safety.test.ts
- commit:a4838ee7ef72f52e52a7b29d1c98c084bb137c44
- commit:167dd6b394a7348c7057395b9d5f543dc4a7c49f
- commit:1bcf7ede23844460ea2eb5ae162d73dd4ce58484
- commit:2ab9c4df27a4d014b91b6abc17920d0ee685f7bd
- commit:47ebc80e2e4a27ad90941456edd0d2cba4c7f423
- commit:99c4f64fe467edd28bc282c36529e86161d94452
- commit:57f32e9f23556c445b983431bc7e49de651232e4
- pr:#740
- pr:#571