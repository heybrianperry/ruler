# Directory-Keyed Agent Registry for Service Boundary Resolution: Agent Interfaces Implementations Types Constants Organized

These rules are ALWAYS ACTIVE for all files in the library layer (`src/lib.ts`) and agent resolution code paths that depend on configuration directory context.

### Rules

- **R-AGENT-001** SHOULD: Agent interfaces, implementations, types, and constants SHOULD be organized into separate modules to enforce boundary separation.
- **R-AGENT-002** MUST: The registry Map MUST be initialized in the library coordination module before any configuration loading logic attempts to register agent collections.
- **R-AGENT-003** MUST: Agent collection retrieval MUST employ nullish coalescing operators for fallback semantics when a configuration directory has no explicitly registered agent collection.
- **R-AGENT-004** SHOULD: Agent resolution logic SHOULD use the central registry rather than direct instantiation at configuration load time.
- **R-AGENT-005** SHOULD: When implementing fallback behavior, add logging or telemetry to track when nullish coalescing returns undefined, enabling detection of missing configuration registrations.
- **R-AGENT-006** MUST: Module boundaries MUST separate agent interfaces, implementations, types, and constants into distinct import paths.
- **R-AGENT-007** SHOULD: Agents SHOULD NOT directly manipulate the registry; maintain clear separation between the registry (service boundary) and agent implementations.

### Verify

```bash
# Locate the library coordination module and verify it contains a Map-based registry
grep -r "Map.*directory\|registry.*Map" src/lib.ts

# Search for registry access patterns and confirm nullish coalescing usage
grep -r "??" src/ | grep -i agent | grep -i registry

# Inspect module import structure for agent interfaces, implementations, types, constants separation
find src/ -name "*agent*" -type f | xargs grep -l "export.*interface\|export.*type\|export.*const\|export.*class" | sort

# Verify no direct agent instantiation bypasses the registry
grep -r "new.*Agent" src/ | grep -v "registry\|Map" | wc -l
```

**Accept when:**
- The registry Map is present in the library layer and uses configuration directory identifiers as keys
- Agent collection retrieval employs nullish coalescing operators for fallback semantics
- Module boundaries separate agent interfaces, implementations, types, and constants into distinct import paths
- Agent resolution uses the central registry rather than direct instantiation
- Logging or telemetry is present to track fallback paths when nullish coalescing returns undefined

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review feedback requiring refactoring to use the registry pattern MUST be applied before merge. Architecture review escalation is required if violations indicate a need to revise the service boundary design.
</enforcement>