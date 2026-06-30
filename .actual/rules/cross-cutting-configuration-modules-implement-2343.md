# Isolate Environment Variable Access for Configuration Discovery: Configuration Modules Implement

These rules are ALWAYS ACTIVE for all modules that read environment variables for application configuration, configuration discovery logic that resolves user-specific or system-wide settings, and runtime initialization code that depends on environment state.

### Rules

- **R-CONFIG-001** MAY: Configuration modules MAY implement caching to avoid repeated process.env lookups for the same variable.
- **R-CONFIG-002** MUST: All direct process.env access for configuration discovery must be isolated to designated configuration modules (e.g., src/core/config.ts) or have documented exceptions.
- **R-CONFIG-003** MUST: Configuration modules must export type-safe accessors with validation for all environment variables.
- **R-CONFIG-004** MUST: Each environment variable must be documented with its purpose, expected format, default value, and security implications.
- **R-CONFIG-005** SHOULD: New environment variable access patterns should use the centralized configuration abstraction rather than direct process.env access.
- **R-CONFIG-006** MUST: Any direct process.env access outside designated configuration modules requires an ESLint disable comment with ticket reference and security team approval for sensitive data.

### Verify

```bash
# Count direct process.env access outside configuration modules
grep -r 'process\.env\.' src/ --exclude-dir=node_modules | grep -v 'src/core/config.ts' | wc -l

# Run configuration tests
npm test -- --testPathPattern=config.test.ts

# Lint for direct process.env access with allowlist
eslint src/ --rule 'no-process-env: error' --ignore-pattern 'src/core/config.ts'
```

**Accept when:**
- All direct process.env access is isolated to designated configuration modules or has documented exceptions with security review approval
- Configuration module includes type-safe accessors with validation for all environment variables
- Unit tests demonstrate configuration behavior can be tested without modifying global process.env
- Each environment variable is documented with purpose, format, default value, and security implications
- ESLint rule no-process-env is enforced in CI pipeline with explicit allowlist for configuration modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules must be checked before accepting configuration module changes. Direct process.env access outside designated modules must be blocked in CI unless explicitly approved by security review.
</enforcement>