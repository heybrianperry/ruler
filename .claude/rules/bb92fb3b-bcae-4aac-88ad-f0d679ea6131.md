<rule_activation id="bb92fb3b-bcae-4aac-88ad-f0d679ea6131" title="Adopt Structured Test Organization with Layered Test Suites: Test Directory Structure" applies_to="tests/**/*.test.ts">
These rules are ALWAYS ACTIVE for all test development and CI/CD pipeline configuration.
</rule_activation>

### Rules

- **R-TSO-001** SHOULD: Test directory structure SHOULD mirror the source code structure within each test layer (unit, integration, e2e).

### Verify

```bash
# Verify test files are organized in layered directories
find tests/ -type f -name '*.test.ts' | grep -E '^tests/(unit|integration|e2e)/' | wc -l

# Check for I/O operations in unit tests (should find none)
grep -r 'describe\|test\|it' tests/unit/ | grep -E '(fetch|http|database|fs\.write)' && echo 'Found I/O in unit tests' || echo 'Unit tests clean'

# Verify test layer directories exist
test -d tests/unit && test -d tests/integration && test -d tests/e2e && echo 'Test directories exist' || echo 'Missing test directories'
```

**Accept when:**
- All test files are located within `tests/unit/`, `tests/integration/`, or `tests/e2e/` directories
- Unit tests contain no external I/O operations (network, database, file system writes)
- CI/CD pipeline configuration includes separate stages or jobs for each test layer
- Test execution reports clearly identify which layer each test belongs to

<enforcement>
Claude Code MUST verify test directory structure compliance before accepting test files. Violations MUST be flagged and remediated according to the layering rules.
</enforcement>