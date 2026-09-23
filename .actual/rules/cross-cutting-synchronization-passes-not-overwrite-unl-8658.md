# Isolation seam for preserving third-party skills and subagents during synchronization: Synchronization Passes Not Overwrite Unlink Delete

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-SYNC-001** MUST_NOT: Synchronization passes MUST NOT overwrite, unlink, or delete native or third-party skill definitions during apply or teardown phases.

### Verify

```bash
# Discover and execute test suites targeting skill propagation, pruning, and subagent preservation
npm test -- tests/skills-propagation.test.ts tests/subagents-propagation.test.ts
```

**Accept when:**
- All skills and subagents propagation tests pass without unlinking non-managed files.
- Symlinks encountered during synchronization are skipped and remain intact.
- Tearing down managed skills leaves existing native and third-party files unmodified.

<enforcement>
Claude Code MUST NOT skip or defer verification. Violation handling: Pull requests altering skill directory cleanup without provenance checks MUST be blocked.
</enforcement>