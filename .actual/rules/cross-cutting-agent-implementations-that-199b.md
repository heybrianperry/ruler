# Standardize Core Library Imports for Agent and Utility Modules: Agent Implementations That

These rules are ALWAYS ACTIVE for all agent implementations in `src/agents/` and core utility modules in `src/core/` that perform file I/O operations or expose public API contracts.

### Rules

- **R-CORE-001** MUST: Agent implementations that perform file I/O MUST import either `fs` or `fs/promises` from Node.js core.
- **R-CORE-002** MUST: All agent implementations MUST extend `AbstractAgent` or implement `IAgent` interface.
- **R-CORE-003** MUST: All agent implementations MUST export their class as a public contract.
- **R-CORE-004** MUST: All `JSON.parse` calls in agent and core utility modules MUST be wrapped in try-catch blocks or equivalent error handling logic.
- **R-CORE-005** SHOULD: New agent implementations SHOULD use `fs/promises` instead of callback-based `fs` to enable async/await patterns.
- **R-CORE-006** SHOULD: Configuration parsing logic SHOULD be centralized in core utility modules rather than duplicated in individual agents.

### Verify

```bash
# Verify path imports in agent and core modules
grep -r "import.*from 'path'" src/agents/ src/core/ | wc -l

# Verify fs/fs-promises imports in agent and core modules
grep -r "import.*from 'fs" src/agents/ src/core/ | wc -l

# Verify agent class exports
grep -r "export.*class.*Agent" src/agents/ | wc -l

# Verify unprotected JSON.parse calls (should return 0)
grep -r "JSON.parse" src/agents/ src/core/ | grep -v "try" | wc -l
```

**Accept when:**
- All agent implementations in `src/agents/` import `path` from Node.js core
- All agent implementations import either `fs` or `fs/promises` from Node.js core
- All agent implementations export their class as a public contract
- All `JSON.parse` calls are wrapped in try-catch blocks or error handling logic
- Core utility modules export named functions for shared functionality (parseFrontmatter, loadSubagentFile, discoverSubagents)

<enforcement>
Clause Code MUST NOT skip or defer verification. CI pipeline MUST fail if agent implementations do not import required Node.js core libraries. Code review MUST block merge if public contracts are not properly exported. Security scan MUST flag unprotected JSON.parse calls for manual review.
</enforcement>