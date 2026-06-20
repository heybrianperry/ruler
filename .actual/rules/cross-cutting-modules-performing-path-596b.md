# Standardize Core Node.js Module Imports for Test Infrastructure and Agent Implementation: Modules Performing Path

These rules are ALWAYS ACTIVE for all test suites, agent implementations, and core infrastructure modules within the Ruler project, including test files under `tests/`, agent implementations under `src/agents/`, core infrastructure modules in `src/core/`, utility modules, and configuration loading operations.

### Rules

- **R-CORE-001** MUST: All modules performing path operations MUST import and use the `path` module for cross-platform path manipulation.
- **R-CORE-002** MUST: All test files MUST use `fs/promises` for file system operations to ensure async-first, non-blocking I/O.
- **R-CORE-003** MUST: Agent implementations MUST NOT use synchronous `fs` operations in production code paths (readFileSync, writeFileSync, etc.).
- **R-CORE-004** SHOULD: Test infrastructure SHOULD use `await fs.mkdtemp(path.join(os.tmpdir(), "ruler-"))` pattern for temporary directory creation.
- **R-CORE-005** SHOULD: CLI integration tests SHOULD use `child_process` with inherited stdio or captured output for consistent command execution.
- **R-CORE-006** MAY: Legacy agent implementations MAY use synchronous fs operations during initialization or constructor execution where async/await is not available, provided this is documented with an exception reference.

### Verify

```bash
# Verify no synchronous fs imports in agents and tests
grep -r "from 'fs'" src/agents/ tests/ | grep -v "fs/promises" | grep -v "fs-extra" || echo 'No synchronous fs imports found'

# Verify path module is imported consistently
grep -r "require('path')\|from 'path'" src/agents/ tests/ | wc -l

# Verify no synchronous fs operations in agent code
grep -r "fs\.readFileSync\|fs\.writeFileSync" src/agents/ || echo 'No synchronous fs operations in agents'

# Run core test suites to verify async patterns work correctly
npm test -- --testNamePattern="(claude-mcp-config|cli-no-mcp|skills-config-precedence)" --passWithNoTests
```

**Accept when:**
- All agent implementations under `src/agents/` import `path` and use `fs/promises` for file operations, with zero synchronous fs calls in production code paths
- All test files under `tests/` import `fs/promises`, `path`, and `os` as needed, with consistent async patterns in beforeEach/afterEach hooks
- Grep verification commands confirm no synchronous fs operations in agent code and consistent core module import patterns across the codebase
- Core test suites pass without errors

<enforcement>
Claude Code MUST NOT skip or defer verification. All four verification commands must pass before accepting changes that modify file system operations or module imports in the specified scope.
</enforcement>