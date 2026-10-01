# Console API Standard Stream Logging: Any External Logging Formatting Dependency Introduced

These rules are ALWAYS ACTIVE for diagnostic, warning, and error reporting across command-line interfaces, core modules, and local file system/configuration workflows.

### Rules

- **R-LOG-001** MUST: If any external logging or formatting dependency is introduced, the consumer MUST inspect the repository lock artifact and resolve the exact locked version before implementation.
- **R-LOG-002** MUST: Route all failure details through designated error formatting helpers prior to emission.
- **R-LOG-003** MUST: Ensure module prefixes are applied consistently across all console warning and error invocations.

### Verify

```bash
# Discover repository test, lint, and verification scripts from the project manifest and execute them
if [ -f "package.json" ]; then
  npm test
  npm run lint
elif [ -f "Cargo.toml" ]; then
  cargo test
  cargo clippy
elif [ -f "go.mod" ]; then
  go test ./...
else
  echo "Please run the project verification and linting commands discovered from your build configuration."
fi
```

**Accept when:**
- All operational errors and warnings are emitted exclusively via console error and warning APIs with standardized prefixes.
- Project verification and linting commands execute without logging-related violations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification via static analysis, lint checks, and test suites is mandatory for all changes touching diagnostic logging and error handling.
</enforcement>