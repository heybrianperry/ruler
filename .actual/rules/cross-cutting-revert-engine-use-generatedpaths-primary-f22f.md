# Revert-engine expanded to track MCP and sidecar provenance at apply-time: Revert Engine Use Generatedpaths Primary Source

These rules are ALWAYS ACTIVE for all apply-engine and revert-engine implementations, particularly code paths that track MCP configuration, sidecar provenance, backup files, and artifact cleanup operations.

### Rules

- **R-REVERT-001** MUST: Revert-engine MUST use generatedPaths as the primary source of truth for identifying artifacts to clean up.
- **R-REVERT-002** MUST: Apply-engine MUST record all artifacts written during apply operations into the generatedPaths collection via trackMcpGeneratedArtifacts or trackOptionalGeneratedArtifacts APIs.
- **R-REVERT-003** MUST: The generatedPaths collection MUST be persisted durably to survive process restarts between apply and revert operations.
- **R-REVERT-004** SHOULD: Revert operations SHOULD be deterministic and based on recorded facts in generatedPaths rather than discovery heuristics.
- **R-REVERT-005** SHOULD: Implementations SHOULD avoid accidental deletion of user files by relying on apply-time accounting rather than pre-existence heuristics.
- **R-REVERT-006** MAY: Future implementations MAY add discovery fallback for artifacts missing from generatedPaths to improve resilience against apply crashes.

### Verify

```bash
# Verify generatedPaths is used as primary cleanup source in revert-engine
grep -n "generatedPaths" src/core/revert-engine.ts | grep -E "(cleanup|remove|delete)" || echo "FAIL: generatedPaths not used for cleanup"

# Verify trackMcpGeneratedArtifacts and trackOptionalGeneratedArtifacts are called during apply
grep -n "track.*GeneratedArtifacts" src/core/apply-engine.ts || echo "FAIL: artifact tracking APIs not invoked"

# Verify generatedPaths persistence mechanism exists
grep -n "persist\|store\|save" src/core/apply-engine.ts | grep -i "generatedPaths" || echo "WARN: generatedPaths persistence not explicitly verified"

# Verify test coverage for MCP provenance tracking
test -f tests/integration/mcp-provenance.test.ts && echo "PASS: MCP provenance tests exist" || echo "FAIL: MCP provenance tests missing"

# Verify revert does not use discovery heuristics as primary mechanism
grep -n "filesystem.*scan\|heuristic.*cleanup" src/core/revert-engine.ts && echo "WARN: discovery heuristics still present" || echo "PASS: primary cleanup uses generatedPaths"
```

**Accept when:**
- generatedPaths is the primary lookup mechanism in revert-engine cleanup logic
- trackMcpGeneratedArtifacts and trackOptionalGeneratedArtifacts are invoked for all artifact writes during apply
- generatedPaths collection is persisted durably between apply and revert operations
- Revert operations succeed based on recorded generatedPaths entries without filesystem discovery fallback
- MCP configuration, backup files, and sidecar artifacts are correctly tracked and cleaned up
- Test coverage in tests/integration/mcp-provenance.test.ts validates end-to-end provenance tracking

<enforcement>
Claude Code MUST NOT skip or defer verification of generatedPaths usage as the primary cleanup source. All apply-engine artifact writes MUST be recorded via the tracking APIs. Revert-engine cleanup MUST rely on generatedPaths as the authoritative record.
</enforcement>