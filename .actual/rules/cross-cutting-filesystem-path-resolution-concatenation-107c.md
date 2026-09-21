# path Module Adoption for Cross-Platform Path Operations: Filesystem Path Resolution Concatenation Normalization Operations

These rules are ALWAYS ACTIVE for all codebase modules performing filesystem path construction, resolution, joining, or normalization, including agent adapters, MCP propagation scripts, revert synchronization engines, and CLI handlers manipulating file references.

### Rules

- **R-PATH-001** MUST: All filesystem path resolution, concatenation, and normalization operations across modules MUST utilize the core path module APIs rather than raw string concatenation or custom delimiter manipulation.

### Verify

```bash
# Discover the project verification script from the repository manifest and execute tests
# Inspect the project linter configuration and run code quality checks for raw path string concatenations
```

**Accept when:**
- All automated test suites pass across all supported host operating systems without path separator failures.
- Static analysis checks complete with zero violations regarding unescaped path delimiters or raw string path concatenations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests containing unapproved raw path string manipulations or platform-specific separator literals are blocked from merging.
</enforcement>