# Standardize Core Node.js Module Imports for Test Infrastructure and Agent Implementation: Modules Not Use

These rules are ALWAYS ACTIVE for all test files under `tests/` directory, all agent implementation files under `src/agents/` directory, core infrastructure modules in `src/core/` that perform file system operations, utility modules (harness.ts, FileSystemUtils.ts), and configuration loading modules (src/revert.ts, src/core/ConfigLoader.ts).

### Rules

- **R-CORE-001** MUST NOT: Production agent code MUST NOT use synchronous file system operations (`fs.readFileSync`, `fs.writeFileSync`) where async alternatives exist.
- **R-CORE-002** MUST: All new test files and agent implementations MUST use `import fs from "fs/promises"` or `import { readFile, writeFile } from "fs/promises"` for file operations.
- **R-CORE-003** MUST: All agent implementations MUST import `path` module for cross-platform path handling.
- **R-CORE-004** MUST: Test infrastructure MUST use async-first patterns with proper `await` in all test lifecycle hooks (beforeEach, afterEach).
- **R-CORE-005** SHOULD: When creating temporary directories in tests, use the pattern `await fs.mkdtemp(path.join(os.tmpdir(), "ruler-"))` as demonstrated in existing test files.
- **R-CORE-006** SHOULD: CLI integration tests SHOULD use `child_process` with inherited stdio or captured output for consistent command execution.

### Verify

```bash
# Verify no synchronous fs imports in agents and tests (excluding fs-extra)
grep -r "from 'fs'" src/agents/ tests/ | grep -v "fs/promises" | grep -v "fs-extra" || echo 'No synchronous fs imports found'

# Verify path module is consistently imported
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
- Core test suites pass without errors, demonstrating async patterns are properly sequenced

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST pass before accepting changes to files in scope. Violations in `src/agents/` MUST cause CI build failure.
</enforcement>