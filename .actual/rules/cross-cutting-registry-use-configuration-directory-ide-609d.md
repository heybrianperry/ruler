# Directory-Keyed Agent Registry for Service Boundary Resolution: Registry Use Configuration Directory Identifiers Keys

These rules are ALWAYS ACTIVE for all agent resolution logic, library layer coordination code, and configuration directory-scoped agent selection paths.

### Rules

- **R-REGISTRY-001** MUST: The registry MUST use configuration directory identifiers as keys for agent collection lookup.

### Verify

```bash
# Locate the library coordination module in the source tree and verify it contains a Map-based registry for directory-to-agent-collection associations
grep -r "Map.*directory\|registry.*Map" src/lib.ts

# Search the codebase for registry access patterns and confirm they use nullish coalescing for fallback behavior
grep -r "registry\.get\|\??\." src/ | grep -E "agent|registry"

# Inspect module import structure to verify separation between agent interfaces, implementations, types, and constants
grep -r "from.*agent" src/ | grep -E "interface|implementation|types|constants"
```

**Accept when:**
- The registry Map is present in the library layer and uses configuration directory identifiers as keys
- Agent collection retrieval employs nullish coalescing operators for fallback semantics
- Module boundaries separate agent interfaces, implementations, types, and constants into distinct import paths

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review verification that agent resolution uses the central registry rather than direct instantiation is mandatory. Static analysis to detect agent collection access patterns that bypass the registry is mandatory. Architecture review to ensure module boundaries between interfaces, implementations, and types are maintained is mandatory.
</enforcement>