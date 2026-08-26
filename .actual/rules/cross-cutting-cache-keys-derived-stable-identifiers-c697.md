# In-Memory Map-Based Cache Layer Pattern: Cache Keys Derived Stable Identifiers That

These rules are ALWAYS ACTIVE for all code implementing caching layers for server connection state, configuration hierarchies, and repeated identifier resolution within operation lifecycles.

### Rules

- **R-CACHE-001** MUST: Cache keys MUST be derived from stable identifiers that uniquely identify the cached resource within the operation scope (URLs for server connections, directory paths for configuration hierarchies, names for server instances).
- **R-CACHE-002** MUST: Use native Map data structures for cache storage and retrieval, leveraging set and get methods for consistent semantics.
- **R-CACHE-003** MUST: Implement cache-check-populate logic consistently: check cache with get, return if present, compute result if absent, populate cache with set, return result.
- **R-CACHE-004** SHOULD: Scope Map instances to operation or request lifecycle where possible to enable automatic cleanup and prevent unbounded memory growth.
- **R-CACHE-005** SHOULD: For module-scoped caches, implement periodic clearing or size-based eviction if unbounded growth is observed.
- **R-CACHE-006** SHOULD: Document cache lifecycle boundaries clearly, including when cached entries become invalid.
- **R-CACHE-007** MAY: Implement explicit invalidation logic when underlying resources (configuration files, server state) change while cached entries remain valid.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the verification script
# to ensure Map-based caching follows consistent patterns
find . -name '.eslintrc*' -o -name 'eslint.config.*' -o -name '.pylintrc' -o -name 'pyproject.toml' | head -1

# Discover the project's test suite and execute tests covering modules with caching
find . -name 'test' -o -name 'tests' -o -name '__tests__' -o -name 'spec' | head -1

# Search for Map instantiation patterns to audit cache scope and key derivation consistency
grep -r "new Map\|Map()" --include="*.js" --include="*.ts" --include="*.py" . 2>/dev/null | grep -v node_modules | head -20

# Verify cache-check-populate pattern usage
grep -r "\.get(\|.set(" --include="*.js" --include="*.ts" . 2>/dev/null | grep -v node_modules | head -20
```

**Accept when:**
- All cache layers use native Map instances with set and get methods for storage and retrieval
- Cache keys are derived from stable identifiers (URLs, paths, names) that uniquely identify the cached resource
- Cache-check-populate logic is implemented consistently: check with get, compute if absent, populate with set
- Map instances are scoped appropriately to operation or request lifecycle, or module-scoped with documented invalidation strategy
- No external caching library dependencies are introduced without documented exception and engineering team approval

<enforcement>
Claude Code MUST NOT skip or defer verification. All new cache implementations MUST be reviewed against R-CACHE-001 through R-CACHE-007 before merge. Violations require code review feedback and refactoring to align with the Map-based caching pattern.
</enforcement>