# Isolation seam for preserving third-party skills and subagents during synchronization: Skill Subagent Synchronization Routines Verify Third

Status: proposed
Date: 2026-09-23
Deciders: AI (signal conversion)

## Context

- Synchronization routines managed by SkillsProcessor and SubagentsProcessor are responsible for applying, updating, and tearing down agent skill and subagent definitions across target environments.
- Previously, synchronization operated via wholesale directory overwrites and indiscriminate pruning passes. This approach purged native agent configurations and third-party artifacts that were not explicitly part of the managed sync payload.
- To prevent silent data loss when re-applying rules or disabling specific skill synchronizations, an isolation seam was introduced via SkillsUtils to distinguish managed skills from native or third-party definitions and symlinks.

## Problem Statement

Wholesale directory synchronization previously wiped native or third-party agent skills, causing silent data loss when re-applying rules or disabling specific agent skill syncs.

## Decision

1. MUST: Skill and subagent synchronization routines MUST verify third-party provenance and preserve unmanaged files when applying or pruning agent skill directories.

## Policy Block

- MUST Skill and subagent synchronization routines MUST verify third-party provenance and preserve unmanaged files when applying or pruning agent skill directories.

In scope:
- SkillsProcessor and SubagentsProcessor execution flows.
- SkillsUtils shared isolation and provenance routines.
- Skill directory apply, propagation, and teardown operations.

Out of scope:
- Internal implementation details of native or third-party skills outside directory synchronization.
- Non-skill agent configuration payloads.

Exceptions:
- ex-1: Synchronization is run in a fully managed sandbox environment where all skills are guaranteed to be transient and non-third-party.

## Rationale

- Preventing the accidental deletion of unmanaged or third-party files avoids destructive data loss during routine synchronization and reconfiguration cycles.
- Skipping symlinks prevents circular references or unintended mutations outside managed directories.
- Centralizing file provenance and isolation logic in SkillsUtils maintains architectural separation between core propagation drivers and filesystem classification rules.

## Consequences

Positive:
- Eliminates silent data loss of third-party and native skills when re-applying or removing rules.
- Allows co-existence of managed and manually configured subagents within the same target environment.
- Centralizes isolation and provenance logic in a shared module.

Negative:
- Increases complexity during pruning operations due to necessary provenance verification steps.
- Requires explicit tracking or heuristic checks to determine whether an artifact is managed or unmanaged.

## Alternatives

- Full overwrite and indiscriminate removal of agent skill directories during propagation passes without checking for native or third-party artifacts. (rejected)
  Rejected because: Caused silent data loss of native or third-party agent skills when rules were re-applied or skill synchronization was disabled.
- Partition managed skills and subagents into a dedicated isolated namespace or external manifest rather than in-place directory checks. (deferred)
  When valid: When target environments enforce strict directory partitioning and do not permit co-located managed and unmanaged artifacts.

## Risks

- Misclassification of managed skills as third-party could lead to orphan artifacts that fail to tear down cleanly.
  Mitigation: Maintain comprehensive unit tests covering propagation and teardown boundary cases for managed versus unmanaged files.
  Owner: Core Engine Team
- Symlink handling variations across target environments could bypass isolation seams.
  Mitigation: Ensure SkillsUtils explicitly checks filesystem attributes before executing read or write operations.
  Owner: Core Engine Team

## Implementation Notes

- Refactored SkillsProcessor and SubagentsProcessor to consume SkillsUtils for detecting managed status.
- Integrated checks to identify and bypass symlinks prior to filesystem mutation.
- Maintained unmanaged subagent and skill files during teardown sequences.

## Continuation Context


Verify commands:
- Discover and execute test suites targeting skill propagation, pruning, and subagent preservation.
- Discover and run static analysis or linters verifying modular separation between processor and utility layers.

Accept when:
- All skills and subagents propagation tests pass without unlinking non-managed files.
- Symlinks encountered during synchronization are skipped and remain intact.
- Tearing down managed skills leaves existing native and third-party files unmodified.

## Enforcement

- Verified by: Automated test suites validating propagation and pruning behavior across mixed native and managed directories.
- Verified by: Code review on modifications to SkillsProcessor, SubagentsProcessor, and SkillsUtils.
- Violation handling: Pull requests altering skill directory cleanup without provenance checks MUST be blocked.
- Violation handling: Test failures identifying inadvertent file deletions during teardown MUST block deployment.
- Exception process: Explicit architectural exceptions approved by the Core Engine Team for environments requiring exclusive single-tenant directory ownership.

## References

- file:src/core/SkillsProcessor.ts
- file:src/core/SubagentsProcessor.ts
- file:src/core/SkillsUtils.ts
- file:src/lib.ts
- tests/skills-propagation.test.ts
- tests/subagents-propagation.test.ts
- commit:6c1c7dc48cfe3a22cabc4e5828431e3a524f532c
- commit:17e5b286db7da0081f1a1cb45e428c305522150f
- commit:3f77fdd3cf39a5a81c69d99b9041cd801ba3c1b6
- commit:775c00ae397620de274a141add12a4eaacd010b0
- commit:3e5f84534f096cad8a752b7cd7b5dd4907a3f954
- commit:47200c6eed17822d1ccbc6337484a9997f907efb
- pr:#734
- pr:#720
- pr:#684