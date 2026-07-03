# Standardize Core Node.js Module Imports for Test Infrastructure and Agent Implementation: Agent Interface Definitions

These rules are ALWAYS ACTIVE for all test files under `tests/` directory, all agent implementation files under `src/agents/` directory, core infrastructure modules in `src/core/`, utility modules (harness.ts, FileSystemUtils.ts), and configuration loading operations in src/revert.ts and src/core/ConfigLoader.ts.

### Rules

- **R-CORE-001** MUST: Agent interface definitions (IAgent, IAgentConfig) MUST be imported from './IAgent' or '../types' to maintain contract consistency.
- **R-CORE-002** MUST: All test files MUST use 'import fs from "fs/promises"' or 'import { readFile, writeFile } from "fs/promises"' for file operations to ensure async-first, non-blocking I/O.
- **R-CORE-003** MUST: All agent implementations MUST import 'path' for cross-platform path handling and MUST use 'fs/promises' for file operations in production code paths.
- **R-CORE-004** MUST: Temporary directories in tests MUST use the pattern 'await fs.mkdtemp(path.join(os.tmpdir(), "ruler-"))' for consistent test isolation.
- **R-CORE-005** MUST: CLI integration tests MUST use 'child_process' with inherited stdio or captured output for consistent command execution.
- **R-CORE-006** SHOULD: All file system operations in test lifecycle hooks (beforeEach, afterEach) SHOULD be properly sequenced with explicit await to prevent race conditions.
- **R-CORE-007** MAY: Synchronous fs operations are permitted only in legacy agent implementations (e.g., GeminiCliAgent) for backward compatibility, documented with EXC-002 exception reference.
- **R-CORE-008** MAY: Synchronous fs operations are permitted during agent initialization or constructor execution where async/await is not available, documented with EXC-001 exception reference.

### Verify

```bash
# Verify no synchronous fs imports in agents and tests (excluding fs/promises and fs-extra)
grep -r "from 'fs'" src/agents/ tests/ | grep -v "fs/promises" | grep -v "fs-extra" || echo 'No synchronous fs imports found'

# Verify path module is consistently imported
grep -r "require('path')\|from 'path'" src/agents/ tests/ | wc -l

# Verify no synchronous fs operations in agent production code
grep -r "fs\.readFileSync\|fs\.writeFileSync" src/agents/ || echo 'No synchronous fs operations in agents'

# Run core test suites to verify async patterns work correctly
npm test -- --testNamePattern="(claude-mcp-config|cli-no-mcp|skills-config-precedence)" --passWithNoTests
```

**Accept when:**
- All agent implementations under src/agents/ import 'path' and use 'fs/promises' for file operations, with zero synchronous fs calls in production code paths
- All test files under tests/ import 'fs/promises', 'path', and 'os' as needed, with consistent async patterns in beforeEach/afterEach hooks
- Grep verification commands confirm no synchronous fs operations in agent code and consistent core module import patterns across the codebase
- Agent interface definitions (IAgent, IAgentConfig) are imported from './IAgent' or '../types' in all agent implementations

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands MUST pass and test suites MUST complete successfully before accepting changes to agent implementations or test infrastructure.
</enforcement>