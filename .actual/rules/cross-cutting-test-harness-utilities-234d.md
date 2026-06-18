# Standardize Core Node.js Module Imports for Test Infrastructure and Agent Implementation: Test Harness Utilities

These rules are ALWAYS ACTIVE for all test files under `tests/` directory, all agent implementation files under `src/agents/` directory, core infrastructure modules in `src/core/`, utility modules (harness.ts, FileSystemUtils.ts), and configuration loading modules (src/revert.ts, src/core/ConfigLoader.ts).

### Rules

- **R-CORE-001** SHOULD: Test harness utilities SHOULD import 'os' for temporary directory creation using os.tmpdir().
- **R-CORE-002** SHOULD: All test files and agent implementations SHOULD import 'fs/promises' for async-first file operations.
- **R-CORE-003** SHOULD: All agent implementations and test utilities SHOULD import 'path' for cross-platform path handling.
- **R-CORE-004** SHOULD: CLI integration tests SHOULD use 'child_process' with inherited stdio or captured output for consistent command execution.
- **R-CORE-005** MUST: Production code in src/agents/ MUST NOT use synchronous fs operations (fs.readFileSync, fs.writeFileSync) except where documented in policy exceptions EXC-001 or EXC-002.
- **R-CORE-006** SHOULD: Temporary directory creation in tests SHOULD follow the pattern 'await fs.mkdtemp(path.join(os.tmpdir(), "ruler-"))'.
- **R-CORE-007** SHOULD: Agent implementations SHOULD follow the pattern in src/agents/AbstractAgent.ts for importing 'path' and defining standard interfaces from './IAgent' or '../types'.

### Verify

```bash
# Verify no synchronous fs imports in agents and tests (excluding fs-extra)
grep -r "from 'fs'" src/agents/ tests/ | grep -v "fs/promises" | grep -v "fs-extra" || echo 'No synchronous fs imports found'

# Count path imports across agents and tests
grep -r "require('path')\|from 'path'" src/agents/ tests/ | wc -l

# Verify no synchronous fs operations in agent code
grep -r "fs\.readFileSync\|fs\.writeFileSync" src/agents/ || echo 'No synchronous fs operations in agents'

# Run core test suites
npm test -- --testNamePattern="(claude-mcp-config|cli-no-mcp|skills-config-precedence)" --passWithNoTests
```

**Accept when:**
- All agent implementations under src/agents/ import 'path' and use 'fs/promises' for file operations, with zero synchronous fs calls in production code paths
- All test files under tests/ import 'fs/promises', 'path', and 'os' as needed, with consistent async patterns in beforeEach/afterEach hooks
- Grep verification commands confirm no synchronous fs operations in agent code and consistent core module import patterns across the codebase
- Core test suites pass without errors

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>