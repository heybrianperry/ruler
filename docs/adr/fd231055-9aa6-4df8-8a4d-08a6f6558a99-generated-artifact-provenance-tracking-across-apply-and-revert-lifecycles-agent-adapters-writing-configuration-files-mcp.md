# Generated artifact provenance tracking across apply and revert lifecycles: Agent Adapters Writing Configuration Files Mcp

Status: proposed
Date: 2026-09-23
Deciders: AI (signal conversion)

## Context

- The apply-engine, revert-engine, and agent adapters interact directly with file system paths when generating configuration files and MCP sidecars. In previous iterations, the apply and revert mechanisms operated with uniform file handling, treating every discovered file on a target path as a preexisting user file.
- This uniform approach led to redundant backups of tool-generated ephemeral files during apply runs and introduced severe risks during revert operations, where user-authored agent configurations could inadvertently be mutated or deleted.
- Introducing explicit provenance registration allows the system to distinguish between tool-generated ephemeral sidecars and user-owned configuration assets.

## Problem Statement

Uniform file handling during apply and revert cycles triggers redundant backups of ephemeral tool sidecars and risks inadvertent deletion or modification of user-authored agent configurations.

## Decision

1. MUST: Agent adapters writing configuration files or MCP sidecars MUST register output file paths with provenance markers prior to writing them to disk.

## Policy Block

- MUST Agent adapters writing configuration files or MCP sidecars MUST register output file paths with provenance markers prior to writing them to disk.

In scope:
- src/core/apply-engine.ts
- src/core/revert-engine.ts
- Agent adapters writing configuration sidecars and MCP configurations

Out of scope:
- Preexisting user-authored configurations managed outside tool lifecycle workflows

Exceptions:
- ex-1: Operating on read-only configuration paths where no mutation or revert capability is exposed.

## Rationale

- Differentiating generated agent outputs from preexisting user files avoids accumulating redundant backups of ephemeral tool artifacts.
- Enforcing provenance verification before mutation or deletion restricts destructive revert actions strictly to verified tool outputs, preserving user-authored configurations.

## Consequences

Positive:
- Eliminates unnecessary file system backups for generated ephemeral sidecars.
- Protects user-authored agent configuration files from accidental deletion during revert routines.
- Establishes a standardized manifest and boundary protocol for all agent adapters generating files.

Negative:
- Agent adapters require explicit provenance registration logic when emitting new configuration artifacts.
- Apply and revert execution cycles require extra verification passes against provenance manifests.

## Alternatives

- Uniform file handling where apply and revert engines treat all discovered files as user files (rejected)
  Rejected because: Triggers redundant backups of ephemeral tool files and risks accidental deletion or modification of user-authored configuration files during revert operations.

## Risks

- A tool-generated artifact missing a provenance marker will be treated as a user file, causing redundant backups and blocking automated reverts.
  Mitigation: Enforce integration test coverage validating provenance registration across all adapter outputs and MCP sidecars.
  Owner: Core Engine Team

## Implementation Notes

- Apply engine logic in src/core/apply-engine.ts coordinates with FileSystemUtils to inspect provenance records before initiating file backups.
- Revert operations in src/core/revert-engine.ts evaluate provenance tags before executing file deletion routines.
- Verify integration behaviors via tests/integration/mcp-provenance.test.ts and tests/integration/generated-sidecar-provenance.test.ts.

## Continuation Context


Verify commands:
- Discover and run the project integration tests for MCP and sidecar provenance handling across apply and revert engines.

Accept when:
- Apply engine avoids backup creation when applying files identified with valid provenance tags.
- Revert engine cleanly reverts generated sidecars and refuses deletion of unverified files.

## Enforcement

- Verified by: Automated integration test suites asserting apply and revert behavior on generated versus user files.
- Verified by: Static review of agent adapter implementations to ensure provenance registration is invoked.
- Violation handling: Pull request checks fail if adapter sidecars are written without registering provenance.
- Violation handling: Apply/revert lifecycle runs abort or fail safety checks when file ownership cannot be validated.
- Exception process: Exceptions for legacy adapters writing untagged files require architecture review approval and an explicit risk assessment.

## References

- file:src/core/apply-engine.ts
- file:src/core/revert-engine.ts
- file:src/core/FileSystemUtils.ts
- file:tests/integration/mcp-provenance.test.ts
- file:tests/integration/generated-sidecar-provenance.test.ts
- commit:eff6c6fe14058193f91208362f2834c2cb550c7e
- commit:7dbe2b55215eb795c00ecadcf3aed4ed3579855d
- commit:1084472aee4f33cf997ae7d652bd993cdfa18aec
- commit:2a0d52e90a7a9effbd41fa24ff0eb0254eb97280
- commit:d72de7ccee64cac85b460c8eca2b5b158f567a9b
- commit:e924d37dbbce05ef7c895ccaabd255ce26b118cc
- pr:#765
- pr:#739
- pr:#733
- pr:#671
- pr:#572