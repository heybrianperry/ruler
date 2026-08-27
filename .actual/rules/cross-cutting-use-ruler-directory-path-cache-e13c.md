# Directory-Keyed Cache as Service Boundary for Agent Resolution: Use Ruler Directory Path Cache Key

These rules are ALWAYS ACTIVE for all configuration loading operations that resolve agent selections from ruler directory configurations, and for subsystems that access both root and nested configuration entries requiring consistent agent resolution.

### Rules

- **R-CACHE-001** MUST: Use the ruler directory path as the cache key when storing and retrieving resolved agent selections.
- **R-CACHE-002** MUST: Initialize the cache Map at the appropriate scope for configuration loading operations, ensuring it is accessible to both cache population and retrieval code paths.
- **R-CACHE-003** MUST: Implement cache population immediately after resolving agents for a ruler directory, before any code attempts to retrieve the cached value.
- **R-CACHE-004** MUST: Use consistent key formatting for ruler directory paths to avoid cache misses due to path normalization differences.
- **R-CACHE-005** SHOULD: Implement cache statistics or metrics to monitor hit rates and identify opportunities for optimization.

### Verify

```bash
# Locate the configuration loading subsystem and identify the cache Map initialization
grep -r "new Map\|new Map()" --include="*.js" --include="*.ts" | grep -i cache

# Trace code paths that populate the cache during configuration entry processing
grep -r "\.set(" --include="*.js" --include="*.ts" | grep -i "ruler\|directory\|path"

# Trace code paths that retrieve cached agent selections
grep -r "\.get(" --include="*.js" --include="*.ts" | grep -i "ruler\|directory\|path"

# Verify cache hits return consistent agent selections
grep -r "cache.*get\|get.*cache" --include="*.js" --include="*.ts" -A 3 | head -20
```

**Accept when:**
- Cache Map is initialized before any configuration loading operations that require agent resolution
- All agent resolution results are stored in the cache using ruler directory paths as keys
- All agent selection retrievals check the cache first before performing resolution
- Cache hits return consistent agent selections for the same ruler directory within a session
- Consistent key formatting is applied to all ruler directory path cache operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-CACHE rules are mandatory and must be verified before accepting configuration loading code that performs agent resolution.
</enforcement>