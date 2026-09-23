# Isolation seam for preserving third-party skills and subagents during synchronization: Skillsprocessor Subagentsprocessor Delegate Isolation Logic Provenance

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-SKILL-001** MUST: SkillsProcessor and SubagentsProcessor MUST delegate isolation logic and provenance inspection to shared utility functions in SkillsUtils.

### Verify

```bash
# Discover and execute test suites targeting skill propagation, pruning, and subagent preservation.
npm test -- tests/skills-propagation.test.ts tests/subagents-propagation.test.ts

# Run static analysis or linters verifying modular separation between processor and utility layers
npm run lint
```

**Accept when:**
- All skills and subagents propagation tests pass without unlinking non-managed files.
- Symlinks encountered during synchronization are skipped and remain intact.
- Tearing down managed skills leaves existing native and third-party files unmodified.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated test suites validating propagation and pruning behavior across mixed native and managed directories must pass, and code review is required on modifications to SkillsProcessor, SubagentsProcessor, and SkillsUtils.
</enforcement>