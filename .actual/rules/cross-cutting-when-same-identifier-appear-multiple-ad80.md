# In-Memory Map-Based Cache Layer Pattern: When Same Identifier Appear Multiple Times

These rules are ALWAYS ACTIVE for all code implementing caching of configuration identifiers, server connections, or agent selections where the same identifier may be queried multiple times within a single operation lifecycle.

### Rules

- **R-CACHE-001** SHOULD: When the same identifier may appear multiple times in a processing loop, the cache SHOULD be consulted before performing resolution, parsing, or instantiation.
- **R-CACHE-002** MUST: Use native Map data structures with `set` and `get` methods for cache storage and retrieval.
- **R-CACHE-003** MUST: Derive cache keys from stable identifiers (URLs, paths, names) that uniquely identify the cached resource.
- **R-CACHE-004** MUST: Implement cache-check-populate logic consistently: check with `get`, compute result if absent, populate cache with `set`, return result.
- **R-CACHE-005** SHOULD: Scope Map instances to operation or request lifecycle where possible to enable automatic cleanup and prevent unbounded memory growth.
- **R-CACHE-006** SHOULD: For module-scoped caches, implement periodic clearing or size-based eviction if unbounded growth is observed.
- **R-CACHE-007** SHOULD: Document cache lifecycle boundaries clearly and implement explicit invalidation when underlying resources change.
- **R-CACHE-008** MAY: Use operation-scoped (function-local Map) for short-lived caching with automatic cleanup, or module-scoped for singleton-like caching where the same instances should be reused across operations.

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
- All cache layers use native Map instances with `set` and `get` methods for storage and retrieval
- Cache keys are derived from stable identifiers (URLs, paths, names) that uniquely identify the cached resource
- Cache-check-populate logic is implemented consistently: check with `get`, compute if absent, populate with `set`
- Map instances are scoped appropriately to operation or request lifecycle where possible
- Cache lifecycle boundaries are documented in code comments or adjacent documentation

<enforcement>
Clause Code MUST NOT skip or defer verification. All new cache implementations MUST be reviewed against these rules before merge. Violations require either alignment with the Map-based pattern or documented exception approval from the engineering team.
</enforcement>