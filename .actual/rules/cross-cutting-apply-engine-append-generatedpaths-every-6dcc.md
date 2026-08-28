# Revert-engine expanded to track MCP and sidecar provenance at apply-time: Apply Engine Append Generatedpaths Every Artifact

These rules are ALWAYS ACTIVE for all apply-engine and revert-engine code paths that write artifacts, manage MCP configurations, create backups, or generate agent-specific sidecars.

### Rules

- **R-APPLY-GEN-001** MUST: Apply-engine MUST append to generatedPaths for every artifact written, before the operation completes.
- **R-APPLY-GEN-002** MUST: The generatedPaths collection MUST be persisted durably to survive process restarts between apply and revert operations.
- **R-APPLY-GEN-003** MUST: All calls to trackMcpGeneratedArtifacts and trackOptionalGeneratedArtifacts MUST occur synchronously before the write operation is considered complete.
- **R-APPLY-GEN-004** SHOULD: Implement idempotent append operations to handle retry scenarios safely.
- **R-APPLY-GEN-005** SHOULD: Monitor generatedPaths collection size and establish limits or cleanup policies for completed operations to prevent unbounded growth.

### Verify

```bash
# Verify that all artifact writes in apply-engine append to generatedPaths
grep -n "trackMcpGeneratedArtifacts\|trackOptionalGeneratedArtifacts" src/core/apply-engine.ts | wc -l

# Verify that generatedPaths is persisted before apply completes
grep -A5 "generatedPaths" src/core/apply-engine.ts | grep -E "persist|save|write|flush"

# Verify test coverage for generatedPaths tracking
grep -l "generatedPaths" tests/unit/core/apply-engine.test.ts tests/integration/mcp-provenance.test.ts

# Verify revert-engine reads from generatedPaths for cleanup
grep -n "generatedPaths" src/core/revert-engine.ts | head -20
```

**Accept when:**
- Every artifact write operation in apply-engine has a corresponding trackMcpGeneratedArtifacts or trackOptionalGeneratedArtifacts call
- generatedPaths is persisted durably before apply operations complete
- Test coverage includes scenarios for MCP configurations, backups, and agent-specific sidecars being tracked
- Revert-engine successfully reads and uses generatedPaths for cleanup operations
- No artifacts are orphaned when apply completes normally

<enforcement>
Claude Code MUST NOT skip or defer verification of generatedPaths tracking. Every artifact write must be auditable through the generatedPaths collection.
</enforcement>