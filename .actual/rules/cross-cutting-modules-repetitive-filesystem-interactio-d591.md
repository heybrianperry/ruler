# path Module Adoption for Cross-Platform Path Operations: Modules Repetitive Filesystem Interactions Encapsulate Composite

These rules are ALWAYS ACTIVE for all codebase modules performing filesystem path construction, resolution, joining, or normalization, including agent adapters, MCP propagation scripts, revert synchronization engines, and CLI handlers manipulating file references.

### Rules

- **R-PATH-001** SHOULD: Modules with repetitive filesystem interactions SHOULD encapsulate composite path operations within designated internal filesystem utility modules rather than duplicating path parsing logic.
- **R-PATH-002** MANDATORY: Locate internal path and filesystem utility modules to reuse existing cross-platform path resolution helpers before implementing custom path logic.
- **R-PATH-003** MANDATORY: When handling paths intended for cross-system serialization or virtual file hierarchies, evaluate whether POSIX-normalized representations are required before persisting.

### Verify

```bash
# Discover and execute project verification script from manifest across supported environments
# Run code quality / static analysis linter to verify no prohibited raw path string concatenations exist
```

**Accept when:**
- All automated test suites pass across all supported host operating systems without path separator failures.
- Static analysis checks complete with zero violations regarding unescaped path delimiters or raw string path concatenations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests containing unapproved raw path string manipulations or platform-specific separator literals are blocked from merging.
</enforcement>