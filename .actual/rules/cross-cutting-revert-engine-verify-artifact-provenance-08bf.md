# Generated artifact provenance tracking across apply and revert lifecycles: Revert Engine Verify Artifact Provenance Ownership

These rules are ALWAYS ACTIVE for all source files, core engines (`src/core/apply-engine.ts`, `src/core/revert-engine.ts`), and agent adapters generating configuration sidecars and MCP configurations.

### Rules

- **R-PROV-001** MUST: The revert engine MUST verify artifact provenance and ownership prior to executing revert deletions or mutations on target files.

### Verify

```bash
# Discover and run the project integration tests for MCP and sidecar provenance handling across apply and revert engines.
npm test -- tests/integration/mcp-provenance.test.ts tests/integration/generated-sidecar-provenance.test.ts
```

**Accept when:**
- Apply engine avoids backup creation when applying files identified with valid provenance tags.
- Revert engine cleanly reverts generated sidecars and refuses deletion of unverified files.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated integration test suites assert apply and revert behavior on generated versus user files, and pull request checks fail if adapter sidecars are written without registering provenance.
</enforcement>