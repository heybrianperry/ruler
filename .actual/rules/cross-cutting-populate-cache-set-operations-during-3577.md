# Directory-Keyed Cache as Service Boundary for Agent Resolution: Populate Cache Set Operations During Configuration

These rules are ALWAYS ACTIVE for all configuration loading subsystems that resolve agent selections from ruler directory configurations and bridge configuration parsing with agent selection logic.

### Rules

- **R-CACHE-001** MUST: Populate the cache using set operations during configuration entry processing, storing the result of agent resolution for each ruler directory.
- **R-CACHE-002** MUST: Initialize the cache Map at the appropriate scope for configuration loading operations, ensuring it is accessible to both cache population and retrieval code paths.
- **R-CACHE-003** MUST: Implement cache population immediately after resolving agents for a ruler directory, before any code attempts to retrieve the cached value.
- **R-CACHE-004** MUST: Use consistent key formatting for ruler directory paths to avoid cache misses due to path normalization differences.
- **R-CACHE-005** SHOULD: Implement cache statistics or metrics to monitor hit rates and identify opportunities for optimization.

### Verify

```bash
# Locate the configuration loading subsystem and verify cache Map initialization
grep -r "new Map\|Map()" --include="*.js" --include="*.ts" | grep -i cache | head -20

# Trace code paths that populate the cache during configuration entry processing
grep -r "cache\.set\|cache\.put" --include="*.js" --include="*.ts" -B 3 -A 3 | grep -i "ruler\|directory\|agent" | head -30

# Trace code paths that retrieve cached agent selections
grep -r "cache\.get\|cache\.has" --include="*.js" --include="*.ts" -B 2 -A 2 | head -30

# Verify cache key format consistency
grep -r "cache\.set\|cache\.get" --include="*.js" --include="*.ts" | grep -o "'[^']*'\|\"[^\"]*\"" | sort | uniq -c | sort -rn
```

**Accept when:**
- Cache Map is initialized before any configuration loading operations that require agent resolution
- All agent resolution results are stored in the cache using ruler directory paths as keys
- All agent selection retrievals check the cache first before performing resolution
- Cache hits return consistent agent selections for the same ruler directory within a session
- Cache key formatting is consistent across all set and get operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All cache population and retrieval operations MUST be traced and verified to use the service boundary pattern. Code review verification and unit/integration tests confirming cache behavior are mandatory before acceptance.
</enforcement>