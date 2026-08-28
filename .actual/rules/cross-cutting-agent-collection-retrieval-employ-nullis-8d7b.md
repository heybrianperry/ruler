# Directory-Keyed Agent Registry for Service Boundary Resolution: Agent Collection Retrieval Employ Nullish Coalescing

These rules are ALWAYS ACTIVE for agent resolution logic that depends on configuration directory context, components in the library layer that coordinate agent selection and configuration, and code paths that require fallback behavior when agent collections are not explicitly registered.

### Rules

- **R-AGENT-REGISTRY-001** MUST: Agent collection retrieval MUST employ nullish coalescing to provide fallback behavior when a directory key has no registered agents.

### Verify

```bash
# Locate the library coordination module in the source tree and verify it contains a Map-based registry for directory-to-agent-collection associations
grep -r "Map.*agent" src/lib.ts || echo "Registry Map not found"

# Search the codebase for registry access patterns and confirm they use nullish coalescing for fallback behavior
grep -r "??" src/ | grep -i agent || echo "Nullish coalescing not found in agent resolution"

# Inspect module import structure to verify separation between agent interfaces, implementations, types, and constants
ls -la src/agents/ | grep -E "(interface|impl|types|constants)" || echo "Module boundary separation not verified"
```

**Accept when:**
- The registry Map is present in the library layer and uses configuration directory identifiers as keys
- Agent collection retrieval employs nullish coalescing operators (`??`) for fallback semantics
- Module boundaries separate agent interfaces, implementations, types, and constants into distinct import paths

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review verification that agent resolution uses the central registry rather than direct instantiation is mandatory. Static analysis to detect agent collection access patterns that bypass the registry is mandatory. Architecture review to ensure module boundaries between interfaces, implementations, and types are maintained is mandatory.
</enforcement>