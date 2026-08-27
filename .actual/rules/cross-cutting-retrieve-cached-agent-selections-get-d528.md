# Directory-Keyed Cache as Service Boundary for Agent Resolution: Retrieve Cached Agent Selections Get Operations

These rules are ALWAYS ACTIVE for all configuration loading operations that resolve agent selections from ruler directory configurations, and for subsystems that access both root and nested configuration entries requiring consistent agent resolution.

### Rules

- **R-CACHE-001** MUST: Retrieve cached agent selections using get operations when accessing root or nested configuration entries.
- **R-CACHE-002** MUST: Initialize the cache Map at the appropriate scope for configuration loading operations, ensuring it is accessible to both cache population and retrieval code paths.
- **R-CACHE-003** MUST: Implement cache population immediately after resolving agents for a ruler directory, before any code attempts to retrieve the cached value.
- **R-CACHE-004** MUST: Use consistent key formatting for ruler directory paths to avoid cache misses due to path normalization differences.
- **R-CACHE-005** SHOULD: Implement cache statistics or metrics to monitor hit rates and identify opportunities for optimization.

### Verify

```bash
# Locate the configuration loading subsystem and identify the cache Map initialization
grep -r "new Map\|Map()" --include="*.js" --include="*.ts" | grep -i "cache\|agent"

# Trace code paths that populate the cache during configuration entry processing
grep -r "\.set(" --include="*.js" --include="*.ts" | grep -i "cache\|agent" | head -20

# Trace code paths that retrieve cached agent selections
grep -r "\.get(" --include="*.js" --include="*.ts" | grep -i "cache\|agent" | head -20

# Verify cache key format consistency for ruler directory paths
grep -r "cache.*key\|key.*format" --include="*.js" --include="*.ts" -A 2 -B 2
```

**Accept when:**
- Cache Map is initialized before any configuration loading operations that require agent resolution
- All agent resolution results are stored in the cache using ruler directory paths as keys
- All agent selection retrievals check the cache first before performing resolution
- Cache hits return consistent agent selections for the same ruler directory within a session
- Ruler directory path keys use consistent formatting across all cache operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All cache operations MUST be traced and verified to use the directory-keyed cache pattern consistently. Code review and unit/integration tests MUST confirm cache population and retrieval behavior before acceptance.
</enforcement>