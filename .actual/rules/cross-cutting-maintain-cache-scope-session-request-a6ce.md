# Directory-Keyed Cache as Service Boundary for Agent Resolution: Maintain Cache Scope Session Request Level

These rules are ALWAYS ACTIVE for all configuration loading subsystems that resolve agent selections from ruler directory configurations and bridge configuration parsing with agent selection logic.

### Rules

- **R-CACHE-001** SHOULD: Maintain cache scope at the session or request level to ensure consistency within a single configuration loading operation.
- **R-CACHE-002** MUST: Initialize the cache Map at the appropriate scope for configuration loading operations, ensuring it is accessible to both cache population and retrieval code paths.
- **R-CACHE-003** MUST: Implement cache population immediately after resolving agents for a ruler directory, before any code attempts to retrieve the cached value.
- **R-CACHE-004** MUST: Use consistent key formatting for ruler directory paths to avoid cache misses due to path normalization differences.
- **R-CACHE-005** SHOULD: Implement cache statistics or metrics to monitor hit rates and identify opportunities for optimization.
- **R-CACHE-006** MUST: Use ruler directory paths as cache keys to create a natural service boundary aligned with the hierarchical configuration structure.
- **R-CACHE-007** MUST: Ensure all agent resolution results are stored in the cache using ruler directory paths as keys.
- **R-CACHE-008** MUST: Ensure all agent selection retrievals check the cache first before performing resolution.
- **R-CACHE-009** MUST: Return consistent agent selections for the same ruler directory within a session.

### Verify

```bash
# Locate the configuration loading subsystem and identify the cache Map initialization
grep -r "new Map\|Map()" --include="*.js" --include="*.ts" | grep -i "cache\|agent" | head -20

# Trace code paths that populate the cache during configuration entry processing
grep -r "\.set(" --include="*.js" --include="*.ts" | grep -i "cache\|agent" | head -20

# Trace code paths that retrieve cached agent selections
grep -r "\.get(" --include="*.js" --include="*.ts" | grep -i "cache\|agent" | head -20

# Verify cache Map is initialized before configuration loading operations
grep -B5 -A5 "loadConfig\|resolveAgent" --include="*.js" --include="*.ts" | grep -i "map\|cache"

# Check for consistent key formatting in cache operations
grep -r "cache\.set\|cache\.get" --include="*.js" --include="*.ts" -A2 -B2 | grep -i "path\|directory"
```

**Accept when:**
- Cache Map is initialized before any configuration loading operations that require agent resolution
- All agent resolution results are stored in the cache using ruler directory paths as keys
- All agent selection retrievals check the cache first before performing resolution
- Cache hits return consistent agent selections for the same ruler directory within a session
- Ruler directory paths use consistent formatting across all cache operations
- Cache scope is limited to session or request level, with explicit initialization and cleanup

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for configuration loading subsystems that resolve agent selections. Code review verification, unit tests, and integration tests MUST confirm cache participation before acceptance. Violations require refactoring to use the cache layer or architecture review escalation.
</enforcement>