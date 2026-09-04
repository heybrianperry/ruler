# Provenance Sidecar Files Track Generated Artifacts Across Apply and Revert Cycles: Mcp Diagnostic Tooling Surface Warning When

These rules are ALWAYS ACTIVE for all generated artifacts produced by the apply engine, including MCP configs, skills, subagent definitions, and settings files that may coexist with user-created equivalents.

### Rules

- **R-PROV-001** MUST: Every generated artifact written by the apply engine MUST have a corresponding `.ruler-provenance.json` sidecar file written in the same transaction or immediately after artifact write, with explicit error handling that rolls back or flags the artifact if the sidecar write fails.
- **R-PROV-002** MUST: The `.ruler-provenance.json` sidecar MUST include the following minimum required fields: `generatingCommit` (SHA string), `generatedAt` (ISO 8601 timestamp), `artifactType` (string enum identifying the artifact category, e.g., 'mcp-config', 'skill', 'subagent'), and `schemaVersion` (integer, starting at 1).
- **R-PROV-003** MUST: The sidecar file path MUST be deterministic and consistent across all callers, using `getGeneratedProvenancePath(artifactPath)` which appends `.ruler-provenance.json` to the artifact's base name (e.g., `mcp-config.json` → `mcp-config.json.ruler-provenance.json`).
- **R-PROV-004** SHOULD: MCP diagnostic tooling SHOULD surface a warning when a generated artifact is found without a corresponding `.ruler-provenance.json` sidecar, indicating a provenance gap that requires remediation.
- **R-PROV-005** MUST: The `isRulerGeneratedFile()` function MUST return false (not throw) when the sidecar file is absent or unparseable, and MUST log a structured warning at the diagnostic level so MCP tooling can surface provenance gaps.
- **R-PROV-006** MUST: The revert engine MUST handle unknown `schemaVersion` values gracefully (e.g., treat as ruler-generated with a warning) rather than throwing a parse error.
- **R-PROV-007** MUST: The revert engine MUST cross-reference the `generatingCommit` SHA in the sidecar against known apply cycles and log a warning before deleting any file whose provenance sidecar references an unrecognized commit.
- **R-PROV-008** MUST: All new generated artifact types added to the apply engine MUST include provenance sidecar writes before code is merged, verified by PR reviewers.
- **R-PROV-009** SHOULD: Apply engine logic SHOULD implement a migration pass that writes provenance sidecars for any existing ruler-generated artifact detected without one, with a diagnostic warning for sidecar-less artifacts in known ruler output paths.
- **R-PROV-010** MUST: `.ruler-provenance.json` MUST be added to default `.gitignore` and `.npmignore` templates to prevent sidecar files from being committed to version control or included in build outputs.

### Verify

```bash
# Verify that all generated artifacts have corresponding provenance sidecars
find . -name '*.ruler-provenance.json' | wc -l

# Verify sidecar schema compliance (schemaVersion, generatingCommit, generatedAt, artifactType present)
find . -name '*.ruler-provenance.json' -exec jq 'has("schemaVersion") and has("generatingCommit") and has("generatedAt") and has("artifactType")' {} \;

# Verify that isRulerGeneratedFile() returns false for artifacts without sidecars
grep -r 'isRulerGeneratedFile' src/core/revert-engine.ts | grep -q 'return false'

# Verify that MCP diagnostic tooling surfaces warnings for provenance gaps
grep -r 'provenance gap' src/ tests/ | grep -q 'warning\|WARN'

# Verify that revert engine handles missing sidecars gracefully
grep -r 'isRulerGeneratedFile' src/core/revert-engine.ts | grep -q 'log.*warning'

# Verify integration test coverage for sidecar survival and revert behavior
grep -r 'generated-sidecar-provenance.test.ts' tests/ | grep -q 'revert\|survive'

# Verify .ruler-provenance.json is in .gitignore
grep -q '\.ruler-provenance\.json' .gitignore
```

**Accept when:**
- All generated artifacts in the apply engine write provenance sidecars atomically with artifact writes
- Sidecar files contain all required fields (generatingCommit, generatedAt, artifactType, schemaVersion)
- `isRulerGeneratedFile()` returns false and logs a warning when sidecar is absent or unparseable
- MCP diagnostic tooling surfaces warnings for provenance gaps
- Revert engine handles missing and unrecognized schema versions gracefully
- Integration tests verify sidecar survival across full revert cycles
- `.ruler-provenance.json` is excluded from version control via .gitignore
- All new artifact types include provenance sidecar writes before merge

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for generated artifact tracking and must be verified before any apply-engine or revert-engine changes are merged.
</enforcement>