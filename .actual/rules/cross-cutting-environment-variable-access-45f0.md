# Isolate Environment Variable Access for Configuration Discovery: Environment Variable Access

These rules are ALWAYS ACTIVE for all modules that read environment variables for application configuration, configuration discovery logic that resolves user-specific or system-wide settings, and runtime initialization code that depends on environment state.

### Rules

- **R-ENV-001** MUST: Environment variable access for configuration discovery MUST be isolated to dedicated configuration modules or utility functions.

### Verify

```bash
# Check for direct process.env access outside designated configuration modules
grep -r 'process\.env\.' src/ --exclude-dir=node_modules | grep -v 'src/core/config.ts' | wc -l

# Run configuration module tests
npm test -- --testPathPattern=config.test.ts

# Lint for process.env access with exceptions for configuration modules
eslint src/ --rule 'no-process-env: error' --ignore-pattern 'src/core/config.ts'
```

**Accept when:**
- All direct process.env access is isolated to designated configuration modules or has documented exceptions
- Configuration module includes type-safe accessors with validation for all environment variables
- Unit tests demonstrate configuration behavior can be tested without modifying global process.env
- ESLint rule no-process-env is enforced in CI pipeline with explicit allowlist for configuration modules
- Code review checklist includes verification that new environment variable access uses configuration abstraction
- Automated tests verify configuration module behavior with various environment states

<enforcement>
Claude Code MUST NOT skip or defer verification of environment variable isolation. All direct process.env access outside designated configuration modules MUST be refactored or documented with explicit exception rationale and security team approval.
</enforcement>