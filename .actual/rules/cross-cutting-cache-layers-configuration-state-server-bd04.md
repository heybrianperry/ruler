# In-Memory Map-Based Cache Layer Pattern: Cache Layers Configuration State Server Connections

These rules are ALWAYS ACTIVE for all code implementing cache layers for configuration state, server connections, and resolved selections within a single process lifecycle.

### Rules

- **R-CACHE-001** MUST: Cache layers for configuration state, server connections, and resolved selections MUST use native Map instances as the primary in-memory storage mechanism.
- **R-CACHE-002** MUST: Implement cache-check-populate logic consistently: check cache with `get()`, return if present, compute result if absent, populate cache with `set()`, return result.
- **R-CACHE-003** MUST: Derive cache keys from stable identifiers (URLs, directory paths, server names) that uniquely identify the cached resource.
- **R-CACHE-004** SHOULD: Scope Map instances to operation or request lifecycle where possible to enable automatic cleanup and prevent unbounded memory growth.
- **R-CACHE-005** SHOULD: For module-scoped caches, implement periodic clearing or size-based eviction if unbounded growth is observed during operation.
- **R-CACHE-006** SHOULD: Document cache lifecycle boundaries clearly, including when cached entries become invalid and whether explicit invalidation is required.
- **R-CACHE-007** MAY: Use operation-scoped (function-local Map) for short-lived caching with automatic cleanup, or module-scoped for singleton-like caching where the same instances should be reused across operations.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the verification script
# to ensure Map-based caching follows consistent patterns
find . -name '.eslintrc*' -o -name 'eslint.config.*' -o -name '.stylelintrc*' | head -1

# Discover the project's test suite and execute tests covering modules with caching
find . -name 'package.json' -o -name 'tsconfig.json' -o -name 'jest.config.*' | head -1

# Search for Map instantiation patterns to audit cache scope and key derivation consistency
grep -r "new Map" --include="*.ts" --include="*.js" | grep -v node_modules | head -20

# Verify cache-check-populate pattern usage
grep -r "\.get(" --include="*.ts" --include="*.js" | grep -v node_modules | head -10
grep -r "\.set(" --include="*.ts" --include="*.js" | grep -v node_modules | head -10
```

**Accept when:**
- All cache layers use native Map instances with `set()` and `get()` methods for storage and retrieval
- Cache keys are derived from stable identifiers (URLs, paths, names) that uniquely identify the cached resource
- Cache-check-populate logic is implemented consistently across all modules: check with `get()`, compute if absent, populate with `set()`
- Map instances are scoped appropriately to operation/request lifecycle or documented as module-scoped with clear invalidation semantics
- No external caching library dependencies are introduced for operation-scoped memoization

<enforcement>
Clause Code MUST NOT skip or defer verification. All new cache implementations MUST be reviewed against R-CACHE-001 through R-CACHE-007 before merge. Static analysis and unit tests MUST confirm Map-based pattern compliance.
</enforcement>