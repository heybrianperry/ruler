# In-Memory Map-Based Cache Layer Pattern: Implementations Declare Map Instances Module Scope

These rules are ALWAYS ACTIVE for all code implementing caching layers for server connection state, configuration hierarchies, and repeated identifier resolution within operation lifecycles.

### Rules

- **R-CACHE-001** MAY: Implementations MAY declare Map instances at module scope for singleton-like caching or at function scope for operation-local caching depending on the required lifecycle.
- **R-CACHE-002** MUST: Cache-check-populate logic MUST follow the pattern: check cache with get, return if present, compute result if absent, populate cache with set, return result.
- **R-CACHE-003** MUST: Cache keys MUST be derived from stable identifiers (URLs, paths, names) that uniquely identify the cached resource.
- **R-CACHE-004** SHOULD: Scope Map instances to operation or request lifecycle where possible to prevent unbounded memory growth.
- **R-CACHE-005** SHOULD: For module-scoped caches, implement periodic clearing or size-based eviction if unbounded growth is observed.
- **R-CACHE-006** MUST: All cache layers MUST use native Map instances with set and get methods for storage and retrieval.
- **R-CACHE-007** SHOULD: Document cache lifecycle boundaries clearly, especially for long-lived caches.
- **R-CACHE-008** SHOULD: For long-lived caches, implement explicit invalidation when underlying resources change.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the verification script
# to ensure Map-based caching follows consistent patterns
find . -name '.eslintrc*' -o -name 'eslint.config.*' -o -name '.stylelintrc*' | head -1

# Discover the project's test suite and execute tests covering modules with caching
find . -name 'package.json' -o -name 'pytest.ini' -o -name 'jest.config.*' | head -1

# Search for Map instantiation patterns to audit cache scope and key derivation consistency
grep -r 'new Map' --include='*.js' --include='*.ts' --include='*.jsx' --include='*.tsx' .

# Verify cache-check-populate pattern usage
grep -r '\.get(' --include='*.js' --include='*.ts' --include='*.jsx' --include='*.tsx' . | grep -E '(cache|Cache|memoiz)'
```

**Accept when:**
- All cache layers use native Map instances with set and get methods for storage and retrieval
- Cache keys are derived from stable identifiers (URLs, paths, names) that uniquely identify the cached resource
- Cache-check-populate logic is implemented consistently: check with get, compute if absent, populate with set
- Module-scoped caches include documentation of their lifecycle and invalidation strategy
- Operation-scoped caches are function-local and cleaned up automatically with function scope

<enforcement>
Claude Code MUST NOT skip or defer verification. All new cache implementations MUST be reviewed against these rules before merge. Static analysis and unit tests MUST confirm compliance with the Map-based caching pattern.
</enforcement>