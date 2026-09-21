# Generated artifact provenance tracking and revert synchronization: Cleanup Revert Engines Refuse Deletion Files

Status: proposed
Date: 2025-05-18
Deciders: AI (signal conversion)

## Context

- Automated distribution of multi-agent rules and tool integrations can inadvertently erase user configs during uninstalls or reverts.
- Explicit provenance tagging and metadata tracking have been introduced for generated MCP sidecars and output files across apply-engine and revert-engine, ensuring rollback operations distinguish managed outputs from user-created configurations.
- Codifying provenance boundaries between generated artifacts and pre-existing files establishes clean reversibility and prevents recursive re-ingestion.

## Problem Statement

How should provenance metadata be standardized across heterogeneous agent config types to prevent third-party agent CLIs from stripping markers and breaking revert detection?

## Decision

1. MUST: Cleanup and revert engines MUST refuse deletion of files lacking verifiable Ruler provenance tags.

## Policy Block

- MUST Cleanup and revert engines MUST refuse deletion of files lacking verifiable Ruler provenance tags.

## Rationale

- Recording verifiable provenance tags allows cleanup engines to distinguish tool-managed outputs from user-authored configurations, preventing accidental data loss during rollback or uninstall operations.
- Establishing strict provenance boundaries eliminates risks of recursive re-ingestion during multi-agent rule synchronization.

## Consequences

Positive:
- Rollback and revert operations cleanly distinguish managed artifacts from user configurations.
- Prevents inadvertent deletion of user-created configurations during uninstall operations.
- Prevents recursive re-ingestion of generated artifacts across agent integrations.

Negative:
- Engine operations must consistently write and parse provenance metadata across all output formats.
- Files lacking verifiable tags will be rejected by cleanup routines, potentially leaving orphaned files if external tools modify or strip metadata.

## Alternatives

- Unconditional deletion based solely on file path tracking without embedded provenance tags (rejected)
  Rejected because: Path-only tracking risks deleting user-modified or pre-existing files during uninstalls or reverts if file ownership cannot be verified.

## Risks

- Third-party agent CLIs may strip markers from heterogeneous config files, breaking revert detection and causing cleanup engines to refuse deletion.
  Mitigation: Standardize provenance metadata tracking across apply-engine and revert-engine and enforce integration tests verifying artifact provenance across supported configurations.
  Owner: Core Engine Team

## Implementation Notes

- Provenance tagging and metadata tracking logic resides within src/core/apply-engine.ts, src/core/revert-engine.ts, and src/revert.ts.
- Verification tests are established in tests/unit/core/apply-engine.test.ts, tests/integration/mcp-provenance.test.ts, and tests/integration/generated-sidecar-provenance.test.ts.

## References

- Commit eff6c6fe14058193f91208362f2834c2cb550c7e
- Commit 7dbe2b55215eb795c00ecadcf3aed4ed3579855d
- Commit 1084472aee4f33cf997ae7d652bd993cdfa18aec
- Commit a012f90962c5da34031980a6447f3bd2aed43795
- Commit d72de7ccee64cac85b460c8eca2b5b158f567a9b
- Commit 167dd6b394a7348c7057395b9d5f543dc4a7c49f
- Commit dcd6434817e1b1c4ce8644c30e46456506aadf35
- Commit 1bcf7ede23844460ea2eb5ae162d73dd4ce58484
- Commit 9d9e0a70288abc0f83a48a8bea5e5318e4aaaad7
- Commit 5f4e772fa6519c80cb57949de2d866123063b006
- Head Commit 0fa2caeef4efaef4c5f8ffbedee8feca4b0b00e2
- PR #571
- PR #576
- PR #681
- PR #733
- PR #739
- PR #741
- PR #744
- PR #765
- src/core/apply-engine.ts
- src/core/revert-engine.ts
- src/revert.ts
- tests/unit/core/apply-engine.test.ts
- tests/integration/mcp-provenance.test.ts
- tests/integration/generated-sidecar-provenance.test.ts