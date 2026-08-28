# Directory-Keyed Agent Registry for Service Boundary Resolution: Agent Collections Resolved Through Map Based

These rules are ALWAYS ACTIVE for all agent resolution logic that depends on configuration directory context, components in the library layer that coordinate agent selection and configuration, and code paths that require fallback behavior when agent collections are not explicitly registered.

### Rules

- **R-DKAR-001** MUST: Agent collections MUST be resolved through a Map-based registry that associates configuration directory identifiers with selected agent collections.
- **R-DKAR-002** MUST: Ensure the registry Map is initialized before any configuration loading logic attempts to register agent collections; consider using a module-level singleton or initialization function.
- **R-DKAR-003** MUST: Maintain clear separation between the registry (service boundary) and agent implementations; agents should not directly manipulate the registry.
- **R-DKAR-004** SHOULD: When implementing fallback behavior, add logging or telemetry to track when nullish coalescing returns undefined, enabling detection of missing configuration registrations.
- **R-DKAR-005** SHOULD: Maintain module boundary separation between agent interfaces, implementations, types, and constants into distinct import paths to enforce architectural layering.

### Verify

```bash
# Locate the library coordination module and verify it contains a Map-based registry
grep -r "Map.*directory\|registry.*Map" src/lib.ts

# Search for registry access patterns and confirm nullish coalescing for fallback
grep -r "??\|\?." src/lib.ts | grep -i agent

# Inspect module import structure to verify separation
grep -r "from.*agent" src/lib.ts | sort | uniq

# Verify agent resolution uses central registry rather than direct instantiation
grep -r "new.*Agent\|Agent(" src/ | grep -v "src/lib.ts" | wc -l
```

**Accept when:**
- The registry Map is present in the library layer and uses configuration directory identifiers as keys
- Agent collection retrieval employs nullish coalescing operators for fallback semantics
- Module boundaries separate agent interfaces, implementations, types, and constants into distinct import paths
- Agent resolution logic is centralized in the library layer rather than scattered across configuration loading code
- Logging or telemetry is present to track fallback paths when registry lookups return undefined

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and must be checked before accepting changes to agent resolution logic.
</enforcement>