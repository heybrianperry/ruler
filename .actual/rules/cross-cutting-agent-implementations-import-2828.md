# Standardize Core Node.js Module Imports for Test Infrastructure and Agent Implementation: Agent Implementations Import

These rules are ALWAYS ACTIVE for all agent implementations under `src/agents/`, test files under `tests/`, core infrastructure modules in `src/core/`, utility modules like `harness.ts` and `FileSystemUtils.ts`, and configuration loading operations in `src/revert.ts` and `src/core/ConfigLoader.ts`.

### Rules

- **R-CORE-IMPORTS-001** MUST: Agent implementations MUST import core modules ('path', 'fs') when implementing applyRulerConfig or configuration management methods.
- **R-CORE-IMPORTS-002** MUST: All test files MUST use 'import fs from "fs/promises"' or 'import { readFile, writeFile } from "fs/promises"' for file operations, never synchronous fs imports.
- **R-CORE-IMPORTS-003** MUST: All file operations in test infrastructure MUST use async/await patterns with 'fs/promises', never synchronous fs calls like readFileSync or writeFileSync.
- **R-CORE-IMPORTS-004** MUST: Agent implementations MUST import 'path' for all path manipulation operations to prevent platform-specific bugs.
- **R-CORE-IMPORTS-005** SHOULD: When creating temporary directories in tests, use the pattern 'await fs.mkdtemp(path.join(os.tmpdir(), "ruler-"))' as demonstrated in existing test files.
- **R-CORE-IMPORTS-006** SHOULD: For CLI integration tests, use 'child_process' with inherited stdio or captured output for consistent command execution.
- **R-CORE-IMPORTS-007** MAY: Legacy agent implementations (e.g., GeminiCliAgent) MAY use synchronous fs operations for backward compatibility with existing configuration formats, provided this is documented with an ADR reference.
- **R-CORE-IMPORTS-008** MAY: Agent implementations MAY require synchronous file system operations during initialization or constructor execution where async/await is not available, provided this is documented with an ADR reference.

### Verify

```bash
# Verify no synchronous fs imports in agents and tests
grep -r "from 'fs'" src/agents/ tests/ | grep -v "fs/promises" | grep -v "fs-extra" || echo 'No synchronous fs imports found'

# Verify path imports are present
grep -r "require('path')\|from 'path'" src/agents/ tests/ | wc -l

# Verify no synchronous fs operations in agent code
grep -r "fs\.readFileSync\|fs\.writeFileSync" src/agents/ || echo 'No synchronous fs operations in agents'

# Run core test suites to verify implementation
npm test -- --testNamePattern="(claude-mcp-config|cli-no-mcp|skills-config-precedence)" --passWithNoTests
```

**Accept when:**
- All agent implementations under `src/agents/` import 'path' and use 'fs/promises' for file operations, with zero synchronous fs calls in production code paths
- All test files under `tests/` import 'fs/promises', 'path', and 'os' as needed, with consistent async patterns in beforeEach/afterEach hooks
- Grep verification commands confirm no synchronous fs operations in agent code and consistent core module import patterns across the codebase
- All new agent implementations follow the pattern in `src/agents/AbstractAgent.ts` for importing 'path' and defining standard interfaces from './IAgent' or '../types'

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST pass before accepting changes to agent implementations, test infrastructure, or core modules that perform file system operations. CI pipeline MUST fail if synchronous fs operations are detected in `src/agents/` directory.
</enforcement>