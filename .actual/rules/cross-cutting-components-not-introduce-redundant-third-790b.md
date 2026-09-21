# path Module Adoption for Cross-Platform Path Operations: Components Not Introduce Redundant Third Party

These rules are ALWAYS ACTIVE for all codebase modules performing filesystem path construction, resolution, joining, or normalization, including agent adapters, MCP propagation scripts, revert synchronization engines, and CLI handlers manipulating file references.

### Rules

- **R-PATH-001** SHOULD NOT: Components introduce redundant third-party path manipulation dependencies when the adopted core path module provides equivalent functionality.
- **R-PATH-002** MANDATORY: Discover the project verification script from the repository manifest and execute the test suite across supported environments to validate path resolution.
- **R-PATH-003** MANDATORY: Inspect the project linter configuration and run the code quality check to verify that no prohibited raw path string concatenations exist.

### Verify

```bash
# Discover and run test suite across environments for path validation
# (Inspect project manifest for test/verification scripts)
if [ -f package.json ]; then
  npm test
elif [ -f Cargo.toml ]; then
  cargo test
elif [ -f pyproject.toml ] || [ -f setup.py ]; then
  pytest
fi

# Run code quality check / linter to verify no raw path concatenations
if [ -f package.json ] && npm run --silent | grep -q "lint"; then
  npm run lint
fi
```

**Accept when:**
- All automated test suites pass across all supported host operating systems without path separator failures.
- Static analysis checks complete with zero violations regarding unescaped path delimiters or raw string path concatenations.

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests containing unapproved raw path string manipulations or platform-specific separator literals are blocked from merging.
</enforcement>