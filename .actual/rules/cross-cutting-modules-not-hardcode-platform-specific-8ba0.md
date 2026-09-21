# path Module Adoption for Cross-Platform Path Operations: Modules Not Hardcode Platform Specific Path

These rules are ALWAYS ACTIVE for all codebase modules performing filesystem path construction, resolution, joining, or normalization, including agent adapters, MCP propagation scripts, revert synchronization engines, and CLI handlers manipulating file references.

### Rules

- **R-PATH-001** MUST_NOT: Modules MUST NOT hardcode platform-specific path separator characters when constructing filesystem paths.

### Verify

```bash
# Discover the project verification script from the repository manifest and execute tests
# Run the project linter configuration to check for prohibited raw path string concatenations
```

**Accept when:**
- All automated test suites pass across all supported host operating systems without path separator failures.
- Static analysis checks complete with zero violations regarding unescaped path delimiters or raw string path concatenations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated continuous integration, static analysis, linting rules prohibiting raw string path concatenations, and peer code review.
</enforcement>