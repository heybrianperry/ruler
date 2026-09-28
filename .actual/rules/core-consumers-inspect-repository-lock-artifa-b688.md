# RuleProcessor Internal Service Boundary Resolution: Consumers Inspect Repository Lock Artifact Verify

These rules are ALWAYS ACTIVE for internal execution engines coordinating agent configurations, output path verification, and modules integrating RuleProcessor for boundary management.

### Rules

- **R-CORE-001** MUST: Consumers MUST inspect the repository lock artifact to verify and resolve the exact locked versions of all external parsing and utility dependencies prior to execution.

### Verify

```bash
# Discover and run the project test suite to validate service boundary isolation and agent lookup behavior
# Discover and run the repository linter and static analysis checks to ensure module boundary rules are respected
```

**Accept when:**
- All discovery verification scripts pass without boundary violations or unhandled exceptions.
- Service lookups and path validations execute predictably without leaking unencapsulated map state.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>