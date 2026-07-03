# Standardize Core Node.js Module Imports for Test Infrastructure and Agent Implementation: Modules Import Additional

These rules are ALWAYS ACTIVE for all test files under `tests/` directory, all agent implementation files under `src/agents/` directory, core infrastructure modules in `src/core/` that perform file system operations, utility modules (harness.ts, FileSystemUtils.ts), and configuration loading modules (src/revert.ts, src/core/ConfigLoader.ts).

### Rules

- **R-CORE-IMPORTS-001** MUST: Use `import fs from "fs/promises"` or `import { readFile, writeFile } from "fs/promises"` for all new test files and agent implementations; synchronous fs operations are prohibited in production agent code paths except where documented exceptions (EXC-001, EXC-002) apply.
- **R-CORE-IMPORTS-002** MUST: Import `path` module in all agent implementations and test files that perform file system operations or path manipulation.
- **R-CORE-IMPORTS-003** MUST: Import `os` module in test infrastructure files when creating temporary directories or accessing platform-specific paths.
- **R-CORE-IMPORTS-004** MUST: Import `child_process` module in CLI integration tests for consistent command execution with inherited or captured stdio.
- **R-CORE-IMPORTS-005** MAY: Modules MAY import additional core modules (`util`, `stream`, `events`) when implementing specialized agent behaviors or test scenarios.
- **R-CORE-IMPORTS-006** MUST: Use async-first patterns with proper `await` sequencing in all test lifecycle hooks (beforeEach, afterEach) to prevent race conditions.
- **R-CORE-IMPORTS-007** MUST: Follow the pattern `await fs.mkdtemp(path.join(os.tmpdir(), "ruler-"))` when creating temporary directories in tests.
- **R-CORE-IMPORTS-008** MUST: Agent implementations follow the pattern in `src/agents/AbstractAgent.ts` for importing `path` and defining standard interfaces from `./IAgent` or `../types`.

### Verify

```bash
# Verify no synchronous fs imports in agents and tests
grep -r "from 'fs'" src/agents/ tests/ | grep -v "fs/promises" | grep -v "fs-extra" || echo 'No synchronous fs imports found'

# Count path imports across agents and tests
grep -r "require('path')\|from 'path'" src/agents/ tests/ | wc -l

# Verify no synchronous fs operations in agent code
grep -r "fs\.readFileSync\|fs\.writeFileSync" src/agents/ || echo 'No synchronous fs operations in agents'

# Run core test suites
npm test -- --testNamePattern="(claude-mcp-config|cli-no-mcp|skills-config-precedence)" --passWithNoTests
```

**Accept when:**
- All agent implementations under `src/agents/` import `path` and use `fs/promises` for file operations, with zero synchronous fs calls in production code paths
- All test files under `tests/` import `fs/promises`, `path`, and `os` as needed, with consistent async patterns in beforeEach/afterEach hooks
- Grep verification commands confirm no synchronous fs operations in agent code and consistent core module import patterns across the codebase
- No third-party file system libraries (fs-extra, etc.) are imported in agent or core infrastructure code

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST pass before accepting changes to agent implementations, test infrastructure, or core modules. Violations in src/agents/ cause CI build failure. Pull requests introducing non-standard core module usage patterns MUST be flagged for architectural review.
</enforcement>