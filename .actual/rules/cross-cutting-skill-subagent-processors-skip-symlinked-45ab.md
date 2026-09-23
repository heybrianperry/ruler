# Isolation seam for preserving third-party skills and subagents during synchronization: Skill Subagent Processors Skip Symlinked Definitions

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-SKILL-001** MUST: Skill and subagent processors MUST skip symlinked definitions during propagation and cleanup passes to maintain isolation.

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
Claude Code MUST NOT skip or defer verification. All modifications to SkillsProcessor, SubagentsProcessor, and SkillsUtils must be reviewed and tested.
</enforcement>