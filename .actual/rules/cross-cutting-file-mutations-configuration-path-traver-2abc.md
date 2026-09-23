# Symlink containment boundary for config resolution and generated outputs: File Mutations Configuration Path Traversals Validate

These rules are ALWAYS ACTIVE for all files matching the configured scope (ConfigLoader, UnifiedConfigLoader, apply-engine, agent adapters, and configuration resolution modules).

### Rules

- **R-SYM-001** MUST: All file mutations and configuration path traversals MUST validate symlink targets against workspace boundaries via FileSystemUtils rather than directly invoking raw fs methods.

### Verify

```bash
# Discover and execute test suites targeting symlink containment, path traversal safety, and configuration loader boundary checks.
npm test -- tests/integration/symlink-output-safety.test.ts
# Static analysis checks verifying that core modules do not import raw fs path manipulation methods directly
npm run lint:fs-imports
```

**Accept when:**
- All integration tests validating symlink output safety pass successfully.
- All path resolution and write operations route strictly through FileSystemUtils containment guards.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests with direct fs method usage in place of FileSystemUtils boundary methods will fail automated linting and code review.
</enforcement>