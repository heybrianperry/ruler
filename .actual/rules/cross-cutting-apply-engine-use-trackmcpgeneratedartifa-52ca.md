# Revert-engine expanded to track MCP and sidecar provenance at apply-time: Apply Engine Use Trackmcpgeneratedartifacts Mcp Related

These rules are ALWAYS ACTIVE for all apply-engine and revert-engine implementations that handle MCP configuration, sidecar artifacts, and provenance tracking during apply and revert operations.

### Rules

- **R-APPLY-MCP-001** SHOULD: Apply-engine SHOULD use trackMcpGeneratedArtifacts for MCP-related artifacts (MCP configuration paths, MCP provenance markers, and agent-specific settings sidecars).
- **R-APPLY-MCP-002** SHOULD: Apply-engine SHOULD use trackOptionalGeneratedArtifacts for conditionally generated artifacts that may or may not be written depending on runtime conditions.
- **R-APPLY-MCP-003** MUST: All artifacts written during apply operations MUST be recorded in the generatedPaths collection at write-time, not discovered at revert-time.
- **R-APPLY-MCP-004** MUST: The generatedPaths collection MUST be persisted durably to survive process restarts between apply and revert operations.
- **R-APPLY-MCP-005** SHOULD: Apply operations SHOULD use idempotent append operations when updating generatedPaths to handle retry scenarios safely.
- **R-APPLY-MCP-006** MUST: Revert-engine MUST use the recorded generatedPaths collection as the authoritative source for cleanup operations, not filesystem discovery heuristics.
- **R-APPLY-MCP-007** SHOULD: Implementations SHOULD monitor apply operation completion rates and investigate partial failures that leave incomplete generatedPaths records.
- **R-APPLY-MCP-008** SHOULD: Implementations SHOULD implement proper locking or serialization for generatedPaths updates to prevent race conditions when multiple apply operations execute concurrently.

### Verify

```bash
# Verify trackMcpGeneratedArtifacts is called for MCP artifacts
grep -r "trackMcpGeneratedArtifacts" src/core/apply-engine.ts

# Verify trackOptionalGeneratedArtifacts is called for conditional artifacts
grep -r "trackOptionalGeneratedArtifacts" src/core/apply-engine.ts

# Verify generatedPaths is persisted durably
grep -r "generatedPaths" src/core/apply-engine.ts | grep -E "(persist|save|write|store)"

# Verify revert-engine reads from generatedPaths
grep -r "generatedPaths" src/core/revert-engine.ts

# Verify test coverage for MCP provenance tracking
test -f tests/integration/mcp-provenance.test.ts && echo "MCP provenance tests exist"

# Verify no discovery-based heuristics in revert cleanup path
! grep -r "filesystem.*scan\|heuristic.*cleanup" src/core/revert-engine.ts || echo "Warning: discovery heuristics still present"
```

**Accept when:**
- trackMcpGeneratedArtifacts is invoked for all MCP configuration paths, provenance markers, and sidecar files written during apply
- trackOptionalGeneratedArtifacts is invoked for all conditionally generated artifacts
- generatedPaths collection is persisted durably and survives process restarts
- revert-engine uses generatedPaths as the authoritative source for cleanup, not filesystem discovery
- All artifacts recorded in generatedPaths are successfully cleaned up during revert operations
- Test coverage includes MCP provenance tracking scenarios and conditional artifact generation
- No orphaned artifacts remain after revert when apply completed successfully
- Concurrent apply operations do not corrupt or lose entries in generatedPaths

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All apply-engine modifications that write MCP artifacts or sidecars MUST use the trackMcpGeneratedArtifacts or trackOptionalGeneratedArtifacts APIs. All revert-engine cleanup logic MUST read from the persisted generatedPaths collection.
</enforcement>