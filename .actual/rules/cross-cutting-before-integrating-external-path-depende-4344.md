# path Module Adoption for Cross-Platform Path Operations: Before Integrating External Path Dependent Packages

These rules are ALWAYS ACTIVE for all codebase modules performing filesystem path construction, resolution, joining, or normalization, including agent adapters, MCP propagation scripts, revert synchronization engines, and CLI handlers manipulating file references.

### Rules

- **R-PATH-001** MUST: Before integrating external path-dependent packages, consumers MUST inspect the project dependency manifest and resolved lock artifact to determine the authoritative locked version and verify API availability against official reference documentation.
- **R-PATH-002** MUST: Locate internal path and filesystem utility modules to reuse existing cross-platform path resolution helpers before implementing custom path logic.

### Verify

```bash
# Discover project dependency manifest, lock artifact, and verification script
# Run test suite across supported environments to validate path resolution
# Run code quality checks to verify no prohibited raw path string concatenations exist
```

**Accept when:**
- All automated test suites pass across all supported host operating systems without path separator failures.
- Static analysis checks complete with zero violations regarding unescaped path delimiters or raw string path concatenations.

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests and code modifications must enforce the path module adoption policy.
</enforcement>