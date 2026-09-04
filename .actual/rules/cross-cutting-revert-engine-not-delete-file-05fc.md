# Provenance Sidecar Files Track Generated Artifacts Across Apply and Revert Cycles: Revert Engine Not Delete File That

These rules are ALWAYS ACTIVE for all revert-engine code paths that perform artifact cleanup and for all apply-engine code paths that generate MCP configs, skills, subagents, and related outputs.

### Rules

- **R-PROV-001** MUST NOT: The revert engine MUST NOT delete a file that lacks a `.ruler-provenance.json` sidecar solely on the basis of its presence in `.ruler-managed.json`, without additional confirmation, to prevent accidental destruction of user files.
- **R-PROV-002** MUST: Every generated artifact (MCP config, skill, subagent, settings file) MUST have a co-located `.ruler-provenance.json` sidecar file written atomically with the artifact itself.
- **R-PROV-003** MUST: The `.ruler-provenance.json` sidecar MUST contain at minimum: `generatingCommit` (SHA string), `generatedAt` (ISO 8601 timestamp), `artifactType` (string enum), and `schemaVersion` (integer, starting at 1).
- **R-PROV-004** MUST: The `getGeneratedProvenancePath(artifactPath)` function MUST return a deterministic sidecar path by appending `.ruler-provenance.json` to the artifact's base name or placing it as a sibling with a consistent naming convention.
- **R-PROV-005** MUST: Apply-engine routines that generate artifacts (e.g., `trackMcpGeneratedArtifacts()`) MUST write the provenance sidecar immediately after successful artifact write, within the same error boundary.
- **R-PROV-006** MUST: The `isRulerGeneratedFile()` function MUST return `false` (not throw) when the sidecar file is absent or unparseable, and MUST log a structured warning at diagnostic level.
- **R-PROV-007** MUST: The revert engine MUST use `isRulerGeneratedFile()` as the primary ownership signal before deleting any artifact, regardless of its presence in `.ruler-managed.json`.
- **R-PROV-008** SHOULD: The revert engine SHOULD log a warning and require explicit confirmation before deleting any file whose provenance sidecar references an unrecognized or stale commit SHA.
- **R-PROV-009** SHOULD: Apply-engine SHOULD implement a migration pass that writes provenance sidecars for any existing ruler-generated artifact detected without one.
- **R-PROV-010** MUST: The revert engine MUST handle unknown `schemaVersion` values in provenance sidecars gracefully (e.g., treat as ruler-generated with a warning) rather than throwing a parse error.

### Verify

```bash
# Verify that revert-engine does not delete files lacking provenance sidecars
grep -r "isRulerGeneratedFile" src/core/revert-engine.ts && echo "✓ Revert engine checks provenance before delete"

# Verify that apply-engine writes sidecars atomically with artifacts
grep -A 5 "trackMcpGeneratedArtifacts" src/core/apply-engine.ts | grep -q "getGeneratedProvenancePath" && echo "✓ Apply engine writes provenance sidecars"

# Verify sidecar schema includes required fields
grep -q "generatingCommit\|generatedAt\|artifactType\|schemaVersion" src/core/FileSystemUtils.ts && echo "✓ Sidecar schema includes required fields"

# Verify integration tests cover sidecar survival and revert behavior
grep -q "sidecar.*revert\|revert.*sidecar" tests/integration/generated-sidecar-provenance.test.ts && echo "✓ Integration tests cover sidecar lifecycle"

# Verify that isRulerGeneratedFile returns false gracefully on missing sidecar
grep -A 10 "isRulerGeneratedFile" src/core/FileSystemUtils.ts | grep -q "return false" && echo "✓ isRulerGeneratedFile handles missing sidecars safely"
```

**Accept when:**
- Revert engine checks `isRulerGeneratedFile()` before deleting any artifact
- Apply engine writes `.ruler-provenance.json` sidecars atomically with artifact writes
- Sidecar JSON includes `generatingCommit`, `generatedAt`, `artifactType`, and `schemaVersion` fields
- `isRulerGeneratedFile()` returns `false` (not error) when sidecar is absent or unparseable
- Integration tests verify sidecar survival across full revert cycles
- Revert engine handles unknown schema versions without throwing parse errors
- No file is deleted solely on the basis of `.ruler-managed.json` presence without provenance confirmation

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-PROV rules are mandatory for artifact safety and must be validated before any apply-engine or revert-engine change is merged.
</enforcement>
