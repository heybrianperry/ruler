# Isolation seam for preserving third-party skills and subagents during synchronization: Skill Subagent Synchronization Routines Verify Third

These rules are ALWAYS ACTIVE for all agent skill and subagent synchronization routines involving `SkillsProcessor`, `SubagentsProcessor`, and `SkillsUtils`.

### Rules

- **R-SKILL-001** MUST: Skill and subagent synchronization routines MUST verify third-party provenance and preserve unmanaged files when applying or pruning agent skill directories.

### Verify

```bash
npm test -- tests/skills-propagation.test.ts tests/subagents-propagation.test.ts
```

**Accept when:**
- All skills and subagents propagation tests pass without unlinking non-managed files.
- Symlinks encountered during synchronization are skipped and remain intact.
- Tearing down managed skills leaves existing native and third-party files unmodified.

<enforcement>
Claude Code MUST NOT skip or defer verification. Violation handling: Pull requests altering skill directory cleanup without provenance checks MUST be blocked. Test failures identifying inadvertent file deletions during teardown MUST block deployment.
</enforcement>