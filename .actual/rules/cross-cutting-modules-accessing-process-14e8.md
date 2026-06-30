# Isolate Environment Variable Access for Configuration Discovery: Modules Accessing Process

These rules are ALWAYS ACTIVE for all modules that read environment variables for application configuration, configuration discovery logic that resolves user-specific or system-wide settings, and runtime initialization code that depends on environment state.

### Rules

- **R-ENV-001** MUST: Modules accessing process.env MUST document which environment variables are read and their expected format.

### Verify

```bash
# Check for undocumented process.env access outside designated configuration modules
grep -r 'process\.env\.' src/ --exclude-dir=node_modules | grep -v 'src/core/config.ts' | wc -l

# Run configuration module tests
npm test -- --testPathPattern=config.test.ts

# Lint for direct process.env access with exceptions for configuration modules
eslint src/ --rule 'no-process-env: error' --ignore-pattern 'src/core/config.ts'
```

**Accept when:**
- All direct process.env access is isolated to designated configuration modules or has documented exceptions
- Configuration module includes type-safe accessors with validation for all environment variables
- Unit tests demonstrate configuration behavior can be tested without modifying global process.env
- Each environment variable is documented with its purpose, expected format, default value, and security implications

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint rule no-process-env is enforced in CI pipeline with explicit allowlist for configuration modules. Code review checklist must verify that new environment variable access uses configuration abstraction. CI pipeline fails on direct process.env access outside designated configuration modules. Pull requests with violations are blocked until refactored or exception is documented with security team approval for sensitive data.
</enforcement>