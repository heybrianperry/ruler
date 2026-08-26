# In-Memory Map-Based Cache Layer Pattern: Cache Population Use Set Method Identifier

These rules are ALWAYS ACTIVE for all code implementing in-memory caching of configuration state, server connections, and agent selections using native Map data structures within a single process lifecycle.

### Rules

- **R-CACHE-001** MUST: Cache population MUST use the set method with the identifier as key and the computed result (server entry, agent array, configuration object) as value.
- **R-CACHE-002** MUST: Cache-check-populate logic MUST follow the pattern: check cache with get, return if present, compute result if absent, populate cache with set, return result.
- **R-CACHE-003** MUST: Cache keys MUST be derived from stable identifiers (URLs, paths, names) that uniquely identify the cached resource.
- **R-CACHE-004** SHOULD: Choose cache scope carefully: operation-scoped (function-local Map) for short-lived caching with automatic cleanup, module-scoped for singleton-like caching where the same instances should be reused across operations.
- **R-CACHE-005** SHOULD: For configuration hierarchies where the same path may be queried multiple times during traversal, cache the resolved result keyed by the canonical path to avoid redundant resolution logic.
- **R-CACHE-006** SHOULD: Scope Map instances to operation or request lifecycle where possible to mitigate memory leak risks from unbounded growth.
- **R-CACHE-007** SHOULD: Document cache lifecycle boundaries clearly and implement explicit invalidation when underlying resources change for long-lived caches.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the verification script
# to ensure Map-based caching follows consistent patterns
find . -name '.eslintrc*' -o -name 'eslint.config.*' -o -name '.pylintrc' -o -name 'pyproject.toml' | head -1

# Discover the project's test suite and execute tests covering modules identified in the evidence
# to verify cache behavior under repeated identifier access
find . -name 'package.json' -o -name 'pytest.ini' -o -name 'setup.py' | xargs grep -l 'test' 2>/dev/null | head -1

# Discover the project's code search capability and search for Map instantiation patterns
# to audit cache scope and key derivation consistency
grep -r 'new Map\|Map()' --include='*.js' --include='*.ts' --include='*.py' . 2>/dev/null | grep -v node_modules | head -20

# Verify cache implementations use set and get methods
grep -r '\.set(\|.get(' --include='*.js' --include='*.ts' --include='*.py' . 2>/dev/null | grep -v node_modules | head -20
```

**Accept when:**
- All cache layers use native Map instances with set and get methods for storage and retrieval
- Cache keys are derived from stable identifiers (URLs, paths, names) that uniquely identify the cached resource
- Cache-check-populate logic is implemented consistently: check with get, compute if absent, populate with set
- Cache scope is documented and appropriate to the operation or module lifecycle
- No external caching library dependencies are introduced without documented exception approval

<enforcement>
Clause Code MUST NOT skip or defer verification. All new cache implementations MUST be reviewed against R-CACHE-001 through R-CACHE-007 before merge. Violations require code review feedback and refactoring to align with the Map-based caching pattern.
</enforcement>