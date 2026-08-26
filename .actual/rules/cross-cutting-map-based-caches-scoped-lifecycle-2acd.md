# In-Memory Map-Based Cache Layer Pattern: Map Based Caches Scoped Lifecycle Operation

These rules are ALWAYS ACTIVE for all code implementing in-memory caching using Map-based data structures for configuration state, server connection management, and operation-scoped memoization.

### Rules

- **R-CACHE-001** SHOULD: Map-based caches SHOULD be scoped to the lifecycle of the operation or request they serve, avoiding unintended cross-request state sharing.
- **R-CACHE-002** MUST: Cache-check-populate logic MUST follow the pattern: check cache with get, return if present, compute result if absent, populate cache with set, return result.
- **R-CACHE-003** MUST: Cache keys MUST be derived from stable identifiers (URLs, paths, names) that uniquely identify the cached resource.
- **R-CACHE-004** SHOULD: Operation-scoped caches (function-local Map) SHOULD be preferred for short-lived caching with automatic cleanup; module-scoped caches SHOULD only be used where the same instances must be reused across operations.
- **R-CACHE-005** MUST: All cache layers MUST use native Map instances with set and get methods for storage and retrieval.
- **R-CACHE-006** SHOULD: For configuration hierarchies where the same path may be queried multiple times during traversal, the resolved result SHOULD be cached keyed by the canonical path to avoid redundant resolution logic.
- **R-CACHE-007** SHOULD: Module-scoped caches SHOULD implement periodic clearing or size-based eviction if unbounded growth is observed to prevent memory leaks.
- **R-CACHE-008** SHOULD: Cache lifecycle boundaries SHOULD be documented clearly; for long-lived caches, explicit invalidation SHOULD be implemented when underlying resources change.

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
- Operation-scoped caches are function-local or request-local, with automatic cleanup at operation boundary
- Module-scoped caches document their scope and include mitigation for unbounded growth
- Cache lifecycle boundaries are documented in code comments or adjacent documentation

<enforcement>
Claude Code MUST NOT skip or defer verification. All new cache implementations MUST be reviewed against R-CACHE-001 through R-CACHE-008 before merge. Violations MUST be addressed via code review feedback requesting alignment with the Map-based caching pattern.
</enforcement>