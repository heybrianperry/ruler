# Symlink containment boundary for config resolution and generated outputs: Core Engines Such Apply Engine Configuration

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-SYM-001** MUST_NOT: Core engines (such as apply-engine) and configuration loaders MUST NOT bypass FileSystemUtils containment guards to perform unconstrained path traversals.

### Verify

```bash
# Discover and execute test suites targeting symlink containment, path traversal safety, and configuration loader boundary checks
npm test -- tests/integration/symlink-output-safety.test.ts
# Static analysis checks verifying that core modules do not import raw fs path manipulation methods directly
npm run lint
```

**Accept when:**
- All integration tests validating symlink output safety pass successfully.
- All path resolution and write operations route strictly through FileSystemUtils containment guards.

<enforcement>
Claude Code MUST NOT skip or defer verification. Violation handling: Pull requests with direct fs method usage in place of FileSystemUtils boundary methods will fail automated linting and code review.
</enforcement>