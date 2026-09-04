# Provenance Sidecar Files Track Generated Artifacts Across Apply and Revert Cycles: Generated Artifacts That Cannot Coexist User

These rules are ALWAYS ACTIVE for all generated artifacts produced by the apply engine, including MCP configs, skills, subagent definitions, and other ruler-owned outputs that may coexist with user-created equivalents.

### Rules

- **R-PROV-001** MUST: Every generated artifact that cannot coexist with user-created equivalents (MCP configs, settings files, skills, subagents) SHALL receive a co-located `.ruler-provenance.json` sidecar file at generation time.
- **R-PROV-002** MUST: The provenance sidecar MUST be written in the same error boundary as the artifact write; if the sidecar write fails, the artifact write MUST be rolled back or explicitly flagged as incomplete.
- **R-PROV-003** MUST: The `.ruler-provenance.json` sidecar MUST contain at minimum: `generatingCommit` (SHA string), `generatedAt` (ISO 8601 timestamp), `artifactType` (string enum), and `schemaVersion` (integer, starting at 1).
- **R-PROV-004** MUST: The sidecar path MUST be deterministic and consistent; use `getGeneratedProvenancePath(artifactPath)` from FileSystemUtils to compute the sidecar location by appending `.ruler-provenance.json` to the artifact's base name.
- **R-PROV-005** MUST: The `isRulerGeneratedFile()` utility MUST return `false` (not throw) when a sidecar is absent or unparseable, and MUST log a structured diagnostic warning so MCP tooling can surface provenance gaps.
- **R-PROV-006** MUST: The revert engine MUST consult the provenance sidecar to determine artifact ownership before deletion; a file is ruler-generated if and only if its corresponding `.ruler-provenance.json` sidecar exists and is parseable.
- **R-PROV-007** MUST: The revert engine MUST handle unknown `schemaVersion` values gracefully (treat as ruler-generated with a warning) rather than throwing a parse error.
- **R-PROV-008** SHOULD: Apply engine routines such as `trackMcpGeneratedArtifacts()` and equivalent functions SHOULD be updated to call `getGeneratedProvenancePath()` and write the sidecar immediately after successful artifact write.
- **R-PROV-009** SHOULD: The `.ruler-provenance.json` sidecar SHOULD include a content hash or commit SHA that the revert engine can cross-reference against known apply cycles to detect user-created files masquerading as ruler-generated.
- **R-PROV-010** SHOULD: `.ruler-provenance.json` patterns SHOULD be added to default `.gitignore` and `.npmignore` templates to prevent sidecar files from being committed to version control.
- **R-PROV-011** MAY: A per-directory `.ruler-provenance-manifest.json` approach MAY be evaluated as a future optimization if per-artifact sidecars produce unacceptable file system overhead in directories with large numbers of generated artifacts.

### Verify

```bash
# Verify that all generated artifacts in known ruler output paths have corresponding .ruler-provenance.json sidecars
find .actual/generated -type f ! -name '*.ruler-provenance.json' | while read artifact; do
  sidecar="${artifact}.ruler-provenance.json"
  if [ ! -f "$sidecar" ]; then
    echo "FAIL: Missing provenance sidecar for $artifact"
    exit 1
  fi
done

# Verify that provenance sidecars contain required fields
find .actual/generated -name '*.ruler-provenance.json' -type f | while read sidecar; do
  for field in generatingCommit generatedAt artifactType schemaVersion; do
    if ! jq -e ".${field}" "$sidecar" > /dev/null 2>&1; then
      echo "FAIL: Missing required field '${field}' in $sidecar"
      exit 1
    fi
  done
done

# Verify that isRulerGeneratedFile() returns false for artifacts without sidecars
test_artifact="/tmp/test-artifact-no-sidecar.json"
echo '{}' > "$test_artifact"
if node -e "const u = require('./src/core/FileSystemUtils'); console.log(u.isRulerGeneratedFile('$test_artifact'))" | grep -q 'true'; then
  echo "FAIL: isRulerGeneratedFile() returned true for artifact without sidecar"
  exit 1
fi
rm -f "$test_artifact"

# Verify that revert engine correctly identifies ruler-generated files via provenance
node -e "const r = require('./src/core/revert-engine'); const fs = require('fs'); const testFile = '/tmp/test-ruler-artifact.json'; const testSidecar = testFile + '.ruler-provenance.json'; fs.writeFileSync(testFile, '{}'); fs.writeFileSync(testSidecar, JSON.stringify({generatingCommit: 'abc123', generatedAt: new Date().toISOString(), artifactType: 'test', schemaVersion: 1})); if (!r.isRulerGeneratedFile(testFile)) { console.error('FAIL: revert engine did not recognize ruler-generated file'); process.exit(1); } fs.unlinkSync(testFile); fs.unlinkSync(testSidecar);"

echo "PASS: All provenance sidecar verification checks passed"
```

**Accept when:**
- All generated artifacts in ruler output paths have corresponding `.ruler-provenance.json` sidecars.
- Every sidecar contains the required fields: `generatingCommit`, `generatedAt`, `artifactType`, and `schemaVersion`.
- `isRulerGeneratedFile()` returns `false` for artifacts without sidecars and logs a diagnostic warning.
- The revert engine correctly identifies and can safely delete ruler-generated files based on provenance sidecars.
- Sidecar writes are atomic with artifact writes; no artifact exists without a corresponding sidecar in the same transaction.
- `.ruler-provenance.json` patterns are present in `.gitignore` and `.npmignore` to prevent version control pollution.
- Integration tests in `generated-sidecar-provenance.test.ts` pass, confirming sidecar survival across full revert cycles.

<enforcement>
Claude Code MUST NOT skip or defer verification. All generated artifacts MUST have provenance sidecars before apply-engine or revert-engine changes are merged. Verification commands MUST pass in CI/CD pipelines.
</enforcement>