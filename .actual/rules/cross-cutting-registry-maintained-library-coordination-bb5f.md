# Directory-Keyed Agent Registry for Service Boundary Resolution: Registry Maintained Library Coordination Layer Centralize

These rules are ALWAYS ACTIVE for all agent resolution logic, library coordination layer code, and configuration-driven agent selection paths that depend on directory context.

### Rules

- **R-REGISTRY-001** SHOULD: The registry SHOULD be maintained in the library coordination layer to centralize agent resolution logic.
- **R-REGISTRY-002** MUST: Ensure the registry Map is initialized before any configuration loading logic attempts to register agent collections; consider using a module-level singleton or initialization function.
- **R-REGISTRY-003** MUST: Maintain clear separation between the registry (service boundary) and agent implementations; agents MUST NOT directly manipulate the registry.
- **R-REGISTRY-004** SHOULD: When implementing fallback behavior, add logging or telemetry to track when nullish coalescing returns undefined, enabling detection of missing configuration registrations.
- **R-REGISTRY-005** MUST: Verify module boundaries separate agent interfaces, implementations, types, and constants into distinct import paths.
- **R-REGISTRY-006** MUST: Agent collection retrieval MUST employ nullish coalescing operators for fallback semantics when no agents are registered for a given directory.

### Verify

```bash
# Locate the library coordination module and verify it contains a Map-based registry
grep -r "Map.*registry\|registry.*Map" src/lib.ts

# Search for registry access patterns and confirm nullish coalescing usage
grep -r "registry.*??\|\??\..*registry" src/

# Inspect module import structure for separation of concerns
grep -E "^import.*from.*/(interfaces|implementations|types|constants)" src/lib.ts

# Verify agent resolution uses central registry rather than direct instantiation
grep -r "new Agent\|Agent(" src/ | grep -v "registry" | wc -l

# Detect agent collection access patterns that bypass the registry
grep -r "agents\[\|agents\.get" src/ | grep -v "registry"
```

**Accept when:**
- The registry Map is present in the library layer (`src/lib.ts`) and uses configuration directory identifiers as keys
- Agent collection retrieval employs nullish coalescing operators (`??`) for fallback semantics
- Module boundaries separate agent interfaces, implementations, types, and constants into distinct import paths
- Agent resolution uses the central registry rather than direct instantiation
- Registry initialization occurs before configuration loading logic attempts to register agent collections
- Logging or telemetry is present to track fallback paths when nullish coalescing returns undefined

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and architecture validation. Violations require refactoring to use the registry pattern or documented exception approval.
</enforcement>