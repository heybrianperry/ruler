# In-Memory Map-Based Cache Layer Pattern: Cache Retrieval Use Get Method Identifier

These rules are ALWAYS ACTIVE for all code implementing in-memory caching of configuration state, server connections, and identifiers that may be queried multiple times within a single operation lifecycle.

### Rules

- **R-CACHE-001** MUST: Cache retrieval MUST use the get method with the identifier as key, checking for existence before performing expensive operations.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the verification script to ensure Map-based caching follows consistent patterns
# Discover the project's test suite and execute tests covering the modules identified in the evidence to verify cache behavior under repeated identifier access
# Discover the project's code search or grep capability and search for Map instantiation patterns to audit cache scope and key derivation consistency
```

**Accept when:**
- All cache layers use native Map instances with set and get methods for storage and retrieval
- Cache keys are derived from stable identifiers (URLs, paths, names) that uniquely identify the cached resource
- Cache-check-populate logic is implemented consistently: check with get, compute if absent, populate with set
- Cache scope is appropriate to operation or request lifecycle to prevent unbounded memory growth
- No external caching library dependencies are introduced for operation-scoped memoization

<enforcement>
Code review MUST verify new cache implementations follow the Map-based pattern with appropriate scope and key derivation before merge. Static analysis or linting rules SHOULD detect cache implementations that deviate from the established pattern. Unit tests MUST cover cache behavior to ensure consistent semantics across modules. Claude Code MUST NOT skip or defer verification.
</enforcement>