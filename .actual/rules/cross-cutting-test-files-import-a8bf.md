# Standardize Core Node.js Module Imports for Test Infrastructure and Agent Implementation: Test Files Import

These rules are ALWAYS ACTIVE for all test files under `tests/` directory, all agent implementation files under `src/agents/` directory, core infrastructure modules in `src/core/`, utility modules (harness.ts, FileSystemUtils.ts), and configuration loading modules (src/revert.ts, src/core/ConfigLoader.ts).

### Rules

- **R-CORE-001** MUST: All test files MUST import `fs/promises` for asynchronous file system operations rather than callback-based `fs` APIs.
- **R-CORE-002** MUST: All agent implementation files MUST import `path` for cross-platform path handling.
- **R-CORE-003** MUST: All file system operations in test infrastructure MUST use async/await patterns with `fs/promises`.
- **R-CORE-004** MUST: Production code in `src/agents/` MUST NOT contain synchronous `fs` operations (readFileSync, writeFileSync, etc.) in primary code paths.
- **R-CORE-005** SHOULD: When creating temporary directories in tests, use the pattern `await fs.mkdtemp(path.join(os.tmpdir(), "ruler-"))`.
- **R-CORE-006** SHOULD: For CLI integration tests, use `child_process` with inherited stdio or captured output for consistent command execution.
- **R-CORE-007** MAY: Legacy agent implementations (e.g., GeminiCliAgent) MAY use synchronous fs operations for backward compatibility, provided they are documented with ADR reference and exception tracking.

### Verify

```bash
# Check for synchronous fs imports in agents and tests
grep -r "from 'fs'" src/agents/ tests/ | grep -v "fs/promises" | grep -v "fs-extra" || echo 'No synchronous fs imports found'

# Verify path imports are present
grep -r "require('path')\|from 'path'" src/agents/ tests/ | wc -l

# Check for synchronous fs operations in agent code
grep -r "fs\.readFileSync\|fs\.writeFileSync" src/agents/ || echo 'No synchronous fs operations in agents'

# Run core test suites to verify async patterns work correctly
npm test -- --testNamePattern="(claude-mcp-config|cli-no-mcp|skills-config-precedence)" --passWithNoTests
```

**Accept when:**
- All agent implementations under `src/agents/` import `path` and use `fs/promises` for file operations, with zero synchronous fs calls in production code paths
- All test files under `tests/` import `fs/promises`, `path`, and `os` as needed, with consistent async patterns in beforeEach/afterEach hooks
- Grep verification commands confirm no synchronous fs operations in agent code and consistent core module import patterns across the codebase
- CI pipeline passes all verification commands without detecting policy violations

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST pass before accepting changes to test files, agent implementations, or core infrastructure modules. Violations in `src/agents/` MUST cause CI build failure.
</enforcement>