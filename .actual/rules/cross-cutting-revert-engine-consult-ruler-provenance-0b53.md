# Provenance Sidecar Files Track Generated Artifacts Across Apply and Revert Cycles: Revert Engine Consult Ruler Provenance Json

These rules are ALWAYS ACTIVE for all code in the apply-engine, revert-engine, and FileSystemUtils modules that generate, track, or consume ruler-generated artifacts.

### Rules

- **R-PROV-001** MUST: The revert engine MUST consult the .ruler-provenance.json sidecar, via isRulerGeneratedFile(), to determine artifact ownership before performing any cleanup or deletion, and MUST NOT rely solely on .ruler-managed.json for this determination.
- **R-PROV-002** MUST: getGeneratedProvenancePath(artifactPath) in FileSystemUtils.ts MUST return the sidecar path by appending .ruler-provenance.json to the artifact's base name or by placing it as a sibling with a deterministic naming convention. The convention MUST be documented and consistent across all callers.
- **R-PROV-003** MUST: The minimum required fields in the .ruler-provenance.json sidecar are: generatingCommit (SHA string), generatedAt (ISO 8601 timestamp), artifactType (string enum identifying the artifact category), and schemaVersion (integer, starting at 1).
- **R-PROV-004** MUST: trackMcpGeneratedArtifacts() and equivalent apply-engine routines MUST write the provenance sidecar immediately after successful artifact write, within the same error boundary.
- **R-PROV-005** MUST: isRulerGeneratedFile() MUST return false (not throw) when the sidecar file is absent or unparseable, and MUST log a structured warning at the diagnostic level so MCP tooling can surface provenance gaps.
- **R-PROV-006** MUST: Sidecar writes MUST be atomic with artifact writes to avoid provenance gaps, requiring careful error handling that rolls back or flags the artifact if the sidecar write fails.
- **R-PROV-007** MUST: The revert engine MUST handle unknown schema versions in provenance sidecars gracefully (e.g., treat as ruler-generated with a warning) rather than throwing a parse error.
- **R-PROV-008** SHOULD: Apply engine logic SHOULD include a migration pass that writes provenance sidecars for any existing ruler-generated artifact detected without one.
- **R-PROV-009** SHOULD: The sidecar SHOULD include a content hash or commit SHA that the revert engine can cross-reference against known apply cycles.
- **R-PROV-010** SHOULD: Revert engine SHOULD log a warning and require explicit confirmation before deleting any file whose provenance sidecar references an unrecognized commit.

### Verify

```bash
# Verify that revert-engine consults isRulerGeneratedFile() before deletion
grep -r "isRulerGeneratedFile" src/core/revert-engine.ts && echo "✓ Revert engine calls isRulerGeneratedFile"

# Verify that getGeneratedProvenancePath is implemented consistently
grep -r "getGeneratedProvenancePath" src/core/FileSystemUtils.ts && echo "✓ Provenance path function exists"

# Verify that apply-engine writes sidecars for MCP artifacts
grep -r "trackMcpGeneratedArtifacts" src/core/apply-engine.ts && echo "✓ Apply engine tracks MCP artifacts"

# Verify that provenance sidecar schema includes required fields
grep -E "generatingCommit|generatedAt|artifactType|schemaVersion" src/core/FileSystemUtils.ts && echo "✓ Sidecar schema fields present"

# Verify integration tests cover sidecar survival and revert behavior
grep -r "generated-sidecar-provenance" tests/integration/ && echo "✓ Integration tests exist"

# Verify that isRulerGeneratedFile handles missing sidecars gracefully
grep -A5 "isRulerGeneratedFile" src/core/FileSystemUtils.ts | grep -E "return false|log.*warning" && echo "✓ Graceful handling of missing sidecars"
```

**Accept when:**
- Revert engine calls isRulerGeneratedFile() before any artifact deletion
- getGeneratedProvenancePath() returns a deterministic sidecar path
- Apply engine writes provenance sidecars atomically with artifacts
- Sidecar JSON includes generatingCommit, generatedAt, artifactType, and schemaVersion
- isRulerGeneratedFile() returns false and logs warnings for missing or unparseable sidecars
- Integration tests verify sidecar survival across full revert cycles
- Revert engine handles unknown schema versions without throwing errors
- All new generated artifact types include provenance sidecar writes

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-PROV rules marked MUST are non-negotiable; SHOULD rules represent best practices that should be followed unless explicitly overridden by project maintainers with documented justification.
</enforcement>
