# process.env for Configuration Directory Resolution: When Utilizing Third Party Parsing Hashing

These rules are ALWAYS ACTIVE for all configuration loading, path resolution, and third-party parsing/hashing routines interacting with environment state.

### Rules

- **R-ENV-001** MUST: When utilizing third-party parsing or hashing libraries within configuration loading, consumers MUST discover the repository dependency manifest and inspect the authoritative lock file to pin exact dependency versions before invocation.
- **R-ENV-002** MUST: Encapsulate environment variable access within dedicated configuration loader methods to prevent proliferation of direct runtime environment reads.
- **R-ENV-003** MUST: Ensure directory fallback logic exists when the specified environment variable is unset or empty.
- **R-ENV-004** MUST: Sanitize and validate directory paths resolved from environment variables before reading filesystem contents.

### Verify

```bash
# Discover and execute the configuration loading test suite using the project test runner
# Discover and run the static analysis security scanner for environment variable handling
```

**Accept when:**
- Configuration resolution tests pass across all supported environments using default and custom path overrides.
- Security analysis reports zero credential leakage or unauthorized environment exposure findings.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>