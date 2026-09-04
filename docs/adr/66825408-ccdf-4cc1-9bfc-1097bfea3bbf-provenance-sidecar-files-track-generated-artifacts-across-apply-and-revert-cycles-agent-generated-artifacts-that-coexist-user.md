# Provenance Sidecar Files Track Generated Artifacts Across Apply and Revert Cycles: Agent Generated Artifacts That Coexist User

Status: proposed
Date: 2025-01-30
Deciders: AI (signal conversion)

## Context

- The ruler system generates artifacts such as MCP configs, skills, and subagent definitions during apply cycles. Prior to this change, a centralized manifest file (.ruler-managed.json) was the sole mechanism for tracking which files were ruler-owned versus user-created. This centralized approach required the revert engine to parse the manifest to determine ownership before performing cleanup.
- Two commits (7dbe2b5, eff6c6f) introduced a distributed provenance model by adding getGeneratedProvenancePath() and isRulerGeneratedFile() to FileSystemUtils. These utilities stamp each generated output with a co-located .ruler-provenance.json sidecar file recording the generating commit and timestamp. Thirteen subsequent commits to apply-engine and revert-engine wired provenance tracking into the full apply/revert lifecycle.
- The apply engine now calls trackMcpGeneratedArtifacts() and equivalent routines to write sidecar files alongside MCP configs, skills, and subagents at generation time. The revert engine reads these sidecars to distinguish ruler-generated artifacts from user files during cleanup, eliminating the need to consult the centralized manifest for ownership decisions.
- Integration tests in generated-sidecar-provenance.test.ts verify that provenance sidecars survive full revert cycles, and MCP diagnostic tests confirm that provenance-based warnings surface correctly. This establishes a durable, distributed data layer for artifact ownership that persists even when the centralized manifest is absent or stale.

## Problem Statement

The centralized .ruler-managed.json manifest is insufficient as the sole ownership signal for generated artifacts: it can become stale, requires parsing during revert, and does not survive scenarios where the manifest itself is removed. A decision is needed on whether provenance sidecars should be mandatory for all generated artifacts or only for those that risk collision with user-created equivalents (e.g., MCP configs, settings files).

## Decision

1. MUST: All agent-generated artifacts that may coexist with user-created equivalents MUST be stamped with a .ruler-provenance.json sidecar file recording at minimum the generating commit SHA and an ISO 8601 timestamp at the time of generation.

## Policy Block

- MUST All agent-generated artifacts that may coexist with user-created equivalents MUST be stamped with a .ruler-provenance.json sidecar file recording at minimum the generating commit SHA and an ISO 8601 timestamp at the time of generation.

## Rationale

- Distributed sidecars are more resilient than a centralized manifest: they survive full reverts because they live alongside their artifacts rather than in a single file that may be removed or corrupted independently.
- Co-locating provenance with the artifact eliminates the need for the revert engine to parse and cross-reference a global manifest, reducing coupling between the revert engine and the manifest format and simplifying ownership resolution to a single file-system lookup.
- Recording the generating commit SHA and timestamp in the sidecar provides an auditable chain of custody, enabling diagnostics to identify which apply cycle produced a given artifact and whether it is stale relative to the current HEAD.
- Scoping the mandatory requirement to artifacts that may coexist with user-created equivalents (MCP configs, settings, skills) addresses the highest-risk collision scenarios while leaving open the possibility of lighter-weight tracking for ruler-exclusive outputs.
- The existing test suite (generated-sidecar-provenance.test.ts, apply-engine.test.ts, revert-engine.test.ts) already validates the core invariants of this model, providing confidence that the rule can be enforced without regressions.

## Consequences

Positive:
- Revert operations become safer: the engine can definitively distinguish ruler-generated files from user files without consulting a potentially stale centralized manifest.
- Provenance sidecars provide an auditable record of which commit generated each artifact, improving debuggability and traceability across apply/revert cycles.
- The distributed model is more resilient to partial failures: loss of .ruler-managed.json does not orphan provenance data for individual artifacts.
- MCP diagnostics gain a reliable signal for surfacing provenance-based warnings, improving operator visibility into the state of generated outputs.
- The ownership contract between apply and revert engines is made explicit and machine-readable at the artifact level rather than inferred from a global list.

Negative:
- Every generated artifact now produces two files on disk (the artifact and its sidecar), increasing file system churn and the number of entries in version control or ignore lists.
- Apply engine logic becomes more complex: sidecar writes must be atomic with artifact writes to avoid provenance gaps, requiring careful error handling.
- Existing generated artifacts produced before this change lack sidecars, creating a migration gap where isRulerGeneratedFile() may return false negatives for legacy outputs until they are regenerated.
- Tooling that scans or lists generated artifacts must be updated to handle sidecar files and avoid treating them as primary artifacts.
- The sidecar format becomes a de facto schema contract; changes to the provenance JSON structure require versioning to avoid breaking revert-engine parsing.

## Alternatives

- Retain centralized .ruler-managed.json as the sole ownership manifest, enhanced with per-artifact commit and timestamp metadata. (rejected)
  Rejected because: A centralized manifest is a single point of failure: it can be removed, corrupted, or become stale independently of the artifacts it tracks. The revert engine must parse the entire manifest to resolve ownership for a single file, and the manifest does not survive scenarios where it is itself a target of revert. The distributed sidecar model directly addresses these failure modes.
- Require provenance sidecars only for artifacts in known collision-risk categories (MCP configs, settings), with no sidecar requirement for ruler-exclusive outputs. (rejected)
  Rejected because: Selective stamping creates an inconsistent ownership signal: the revert engine cannot apply a uniform isRulerGeneratedFile() check and must maintain a separate list of artifact categories exempt from provenance. This reintroduces category-level coupling and increases the risk of future artifacts being incorrectly classified as user files.
  When valid: Acceptable as a transitional policy during migration if the full sidecar rollout is phased, provided that ruler-exclusive paths are explicitly enumerated and enforced.
- Embed provenance metadata inside the generated artifact itself (e.g., a comment header or JSON field) rather than in a separate sidecar file. (rejected)
  Rejected because: Embedding provenance in the artifact requires format-specific parsing logic for each artifact type (JSON, YAML, shell scripts, etc.) and risks corrupting or invalidating the artifact if the embedding is malformed. A uniform sidecar format decouples provenance from artifact content and allows a single isRulerGeneratedFile() implementation regardless of artifact type.
  When valid: May be appropriate as a supplementary signal for artifact types where inline metadata is idiomatic (e.g., generated code files with header comments), but should not replace the sidecar.
- Use a per-directory .ruler-provenance-manifest.json that tracks all ruler-generated files within that directory subtree. (deferred)
  When valid: Could be evaluated if the per-artifact sidecar approach produces unacceptable file system overhead in directories with large numbers of generated artifacts. Would require a new directory-scoped ownership resolution strategy in the revert engine.

## Risks

- Legacy generated artifacts produced before provenance sidecars were introduced lack .ruler-provenance.json files, causing isRulerGeneratedFile() to return false negatives and potentially leaving orphaned files after revert.
  Mitigation: Implement a migration pass in the apply engine that writes provenance sidecars for any existing ruler-generated artifact detected without one. Document the migration requirement and add a diagnostic warning for sidecar-less artifacts in known ruler output paths.
  Owner: apply-engine maintainers
- Non-atomic sidecar writes (artifact written but sidecar write fails) leave artifacts without provenance, creating ownership ambiguity for the revert engine.
  Mitigation: Apply engine MUST write the sidecar in the same transaction or immediately after the artifact write with explicit error handling that rolls back or flags the artifact if the sidecar write fails. Integration tests should cover partial-write failure scenarios.
  Owner: apply-engine maintainers
- The .ruler-provenance.json sidecar schema evolves without versioning, breaking revert-engine parsing for sidecars written by older apply-engine versions.
  Mitigation: Include a schemaVersion field in the sidecar JSON from the initial implementation. The revert engine MUST handle unknown schema versions gracefully (e.g., treat as ruler-generated with a warning) rather than throwing a parse error.
  Owner: FileSystemUtils / revert-engine maintainers
- Sidecar files are inadvertently committed to version control or included in build outputs, polluting repositories and CI artifacts.
  Mitigation: Add .ruler-provenance.json to default .gitignore and .npmignore templates generated by ruler. Document the sidecar file pattern in onboarding materials so teams can add appropriate ignore rules.
  Owner: ruler project maintainers
- A user manually creates a .ruler-provenance.json file alongside their own artifact, causing the revert engine to incorrectly classify a user file as ruler-generated and delete it.
  Mitigation: The sidecar SHOULD include a content hash or commit SHA that the revert engine can cross-reference against known apply cycles. Revert engine SHOULD log a warning and require explicit confirmation before deleting any file whose provenance sidecar references an unrecognized commit.
  Owner: revert-engine maintainers

## Implementation Notes

- getGeneratedProvenancePath(artifactPath) in FileSystemUtils.ts MUST return the sidecar path by appending .ruler-provenance.json to the artifact's base name (e.g., mcp-config.json → mcp-config.json.ruler-provenance.json) or by placing it as a sibling with a deterministic naming convention. The convention MUST be documented and consistent across all callers.
- The minimum required fields in the .ruler-provenance.json sidecar are: generatingCommit (SHA string), generatedAt (ISO 8601 timestamp), artifactType (string enum identifying the artifact category, e.g., 'mcp-config', 'skill', 'subagent'), and schemaVersion (integer, starting at 1).
- trackMcpGeneratedArtifacts() and equivalent apply-engine routines MUST be updated to call getGeneratedProvenancePath() and write the sidecar immediately after successful artifact write, within the same error boundary.
- isRulerGeneratedFile() MUST return false (not throw) when the sidecar file is absent or unparseable, and MUST log a structured warning at the diagnostic level so MCP tooling can surface provenance gaps.
- The integration test generated-sidecar-provenance.test.ts MUST be extended to cover: (a) sidecar survival across a full revert cycle, (b) revert engine behavior when sidecar is absent, and (c) revert engine behavior when sidecar schemaVersion is unrecognized.
- PRs 738, 739, 744, and 765 contain the reference implementation. Reviewers of future apply-engine or revert-engine changes MUST verify that any new generated artifact type includes provenance sidecar writes before merging.

## References

- src/core/FileSystemUtils.ts
- src/core/apply-engine.ts
- src/core/revert-engine.ts
- tests/integration/generated-sidecar-provenance.test.ts
- tests/unit/core/apply-engine.test.ts
- tests/unit/core/revert-engine.test.ts
- commit:7dbe2b55215eb795c00ecadcf3aed4ed3579855d
- commit:eff6c6fe14058193f91208362f2834c2cb550c7e
- commit:9d9e0a70288abc0f83a48a8bea5e5318e4aaaad7
- commit:f16bebcb9d9187c44d886acc70f08f5ec7870e6d
- commit:0fa2caeef4efaef4c5f8ffbedee8feca4b0b00e2 (HEAD)
- PR#738
- PR#739
- PR#744
- PR#765
- symbol:getGeneratedProvenancePath
- symbol:isRulerGeneratedFile
- symbol:trackMcpGeneratedArtifacts