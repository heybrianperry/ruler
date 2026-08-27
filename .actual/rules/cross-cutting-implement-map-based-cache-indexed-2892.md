# Directory-Keyed Cache as Service Boundary for Agent Resolution: Implement Map Based Cache Indexed Ruler

These rules are ALWAYS ACTIVE for all configuration loading subsystems that resolve agent selections from ruler directory configurations, particularly code that bridges configuration parsing and agent selection logic across root and nested configuration entries.

### Rules

- **R-CACHE-001** MUST: Implement a Map-based cache indexed by ruler directory paths at the service boundary between configuration loading and agent resolution subsystems.
- **R-CACHE-002** MUST: Initialize the cache Map at the appropriate scope for configuration loading operations, ensuring it is accessible to both cache population and retrieval code paths.
- **R-CACHE-003** MUST: Implement cache population immediately after resolving agents for a ruler directory, before any code attempts to retrieve the cached value.
- **R-CACHE-004** MUST: Use consistent key formatting for ruler directory paths to avoid cache misses due to path normalization differences.
- **R-CACHE-005** SHOULD: Implement cache statistics or metrics to monitor hit rates and identify opportunities for optimization.
- **R-CACHE-006** SHOULD: Add logging or debugging hooks to track cache hits/misses and cache population events.

### Verify

```bash
# Locate the configuration loading subsystem and verify cache Map initialization
grep -r "new Map\|new Map()\|Map<" --include="*.ts" --include="*.js" | grep -i "cache\|ruler\|agent"

# Trace code paths that populate the cache during configuration entry processing
grep -r "\.set(" --include="*.ts" --include="*.js" | grep -i "cache\|ruler\|directory"

# Trace code paths that retrieve cached agent selections
grep -r "\.get(" --include="*.ts" --include="*.js" | grep -i "cache\|ruler\|agent"

# Verify cache keys use ruler directory paths consistently
grep -r "cache\.set\|cache\.get" --include="*.ts" --include="*.js" -A 2 -B 2
```

**Accept when:**
- Cache Map is initialized before any configuration loading operations that require agent resolution
- All agent resolution results are stored in the cache using ruler directory paths as keys
- All agent selection retrievals check the cache first before performing resolution
- Cache hits return consistent agent selections for the same ruler directory within a session
- Ruler directory path keys use consistent formatting across all cache operations
- Cache population occurs immediately after agent resolution, before retrieval attempts

<enforcement>
Claude Code MUST NOT skip or defer verification. All cache operations MUST be traced to confirm they use the Map-based cache indexed by ruler directory paths. Code review verification and unit/integration tests confirming cache behavior are mandatory before acceptance.
</enforcement>