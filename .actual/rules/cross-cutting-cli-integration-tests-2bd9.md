# Standardize Core Node.js Module Imports for Test Infrastructure and Agent Implementation: Cli Integration Tests

These rules are ALWAYS ACTIVE for all test files under `tests/` directory, all agent implementation files under `src/agents/` directory, core infrastructure modules in `src/core/`, utility modules (harness.ts, FileSystemUtils.ts), and configuration loading modules (src/revert.ts, src/core/ConfigLoader.ts).

### Rules

- **R-CORE-001** SHOULD: CLI integration tests SHOULD import 'child_process' for spawning Ruler CLI commands with inherited or captured stdio.
- **R-CORE-002** SHOULD: All test files SHOULD import 'fs/promises' for async-first file operations, preventing blocking operations during test execution.
- **R-CORE-003** SHOULD: All agent implementations SHOULD import 'path' to prevent platform-specific path handling bugs (Windows vs. Unix).
- **R-CORE-004** SHOULD: Test infrastructure SHOULD use 'await fs.mkdtemp(path.join(os.tmpdir(), "ruler-"))' pattern for creating temporary directories.
- **R-CORE-005** MUST: Production code in src/agents/ MUST NOT use synchronous fs operations (fs.readFileSync, fs.writeFileSync) in non-initialization code paths.
- **R-CORE-006** SHOULD: Agent implementations SHOULD follow the pattern in src/agents/AbstractAgent.ts for importing 'path' and defining standard interfaces from './IAgent' or '../types'.
- **R-CORE-007** MAY: Synchronous file system operations MAY be used during agent initialization or constructor execution where async/await is not available (EXC-001).
- **R-CORE-008** MAY: Legacy agent implementations MAY use synchronous fs operations for backward compatibility with existing configuration formats (EXC-002).

### Verify

```bash
# Verify no synchronous fs imports in agents and tests (excluding fs-extra)
grep -r "from 'fs'" src/agents/ tests/ | grep -v "fs/promises" | grep -v "fs-extra" || echo 'No synchronous fs imports found'

# Verify path module is consistently imported
grep -r "require('path')\|from 'path'" src/agents/ tests/ | wc -l

# Verify no synchronous fs operations in agent production code
grep -r "fs\.readFileSync\|fs\.writeFileSync" src/agents/ || echo 'No synchronous fs operations in agents'

# Run integration tests to verify child_process usage in CLI tests
npm test -- --testNamePattern="(claude-mcp-config|cli-no-mcp|skills-config-precedence)" --passWithNoTests
```

**Accept when:**
- All agent implementations under src/agents/ import 'path' and use 'fs/promises' for file operations, with zero synchronous fs calls in production code paths
- All test files under tests/ import 'fs/promises', 'path', and 'os' as needed, with consistent async patterns in beforeEach/afterEach hooks
- Grep verification commands confirm no synchronous fs operations in agent code and consistent core module import patterns across the codebase
- CLI integration tests successfully spawn Ruler CLI commands using child_process with inherited or captured stdio

<enforcement>
Clause Code MUST NOT skip or defer verification. ESLint rules MUST enforce 'fs/promises' imports and flag synchronous fs operations in production code. CI pipeline MUST run grep-based verification commands to detect policy violations before merge. Build MUST fail if synchronous fs operations are detected in src/agents/ directory.
</enforcement>