# Directory-Keyed Cache as Service Boundary for Agent Resolution: Implement Fallback Handling Cache Misses Nullish

These rules are ALWAYS ACTIVE for all configuration loading subsystems that resolve agent selections from ruler directory configurations and bridge configuration parsing with agent selection logic.

### Rules

- **R-CACHE-001** SHOULD: Implement fallback handling for cache misses using nullish coalescing or equivalent patterns to handle directories not yet resolved.
- **R-CACHE-002** MUST: Initialize the cache Map at the appropriate scope for configuration loading operations, ensuring it is accessible to both cache population and retrieval code paths.
- **R-CACHE-003** MUST: Implement cache population immediately after resolving agents for a ruler directory, before any code attempts to retrieve the cached value.
- **R-CACHE-004** MUST: Use consistent key formatting for ruler directory paths to avoid cache misses due to path normalization differences.
- **R-CACHE-005** SHOULD: Implement cache statistics or metrics to monitor hit rates and identify opportunities for optimization.
- **R-CACHE-006** MUST: Check the cache first before performing agent resolution in all agent selection retrieval code paths.

### Verify

```bash
# Locate the configuration loading subsystem and verify cache Map initialization
grep -r "new Map\|Map()" --include="*.js" --include="*.ts" | grep -i cache

# Trace code paths that populate the cache during configuration entry processing
grep -r "\.set(" --include="*.js" --include="*.ts" | grep -i "ruler\|directory\|agent"

# Trace code paths that retrieve cached agent selections
grep -r "\.get(" --include="*.js" --include="*.ts" | grep -i "ruler\|directory\|agent"

# Verify fallback handling with nullish coalescing
grep -r "\?\?" --include="*.js" --include="*.ts" | grep -i "cache\|agent"

# Check for consistent key formatting in cache operations
grep -r "normalize\|Path\|resolve" --include="*.js" --include="*.ts" | grep -i "key\|cache"
```

**Accept when:**
- Cache Map is initialized before any configuration loading operations that require agent resolution
- All agent resolution results are stored in the cache using ruler directory paths as keys
- All agent selection retrievals check the cache first before performing resolution
- Cache hits return consistent agent selections for the same ruler directory within a session
- Fallback handling using nullish coalescing or equivalent patterns is present for cache misses
- Ruler directory path keys use consistent formatting across all cache operations

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review verification that configuration loading code uses the cache Map for all agent resolution operations is mandatory. Unit tests verifying cache population and retrieval behavior with multiple ruler directories are mandatory. Integration tests confirming consistent agent selection for the same directory across root and nested configurations are mandatory.
</enforcement>