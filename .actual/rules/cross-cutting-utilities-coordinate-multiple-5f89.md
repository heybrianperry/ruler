# Standardize Asynchronous Configuration Loading Pattern for Agent Initialization: Utilities Coordinate Multiple

These rules are ALWAYS ACTIVE for all agent class implementations, core utility modules for configuration loading, and functions that read agent settings, MCP configurations, and YAML/TOML subagent definitions during initialization and setup methods in agent lifecycle.

### Rules

- **R-ASYNC-001** MAY: Utilities MAY coordinate multiple async operations using Promise.all or sequential awaits based on dependency requirements.
- **R-ASYNC-002** MUST: All agent configuration loading functions use async/await with fs/promises and avoid synchronous fs operations in agent initialization paths.
- **R-ASYNC-003** MUST: Use fs/promises import for all new configuration loading code; prefer `import { readFile } from "fs/promises"` over promisify(fs.readFile).
- **R-ASYNC-004** SHOULD: Structure configuration loading functions to return parsed objects rather than raw strings, encapsulating both I/O and parsing concerns.
- **R-ASYNC-005** SHOULD: Add JSDoc annotations to async configuration functions documenting expected file formats, error conditions, and return types.
- **R-ASYNC-006** MUST: In agent constructors or initialization methods, ensure all async configuration loading completes before marking agent as ready/initialized.
- **R-ASYNC-007** SHOULD: Implement comprehensive error handling with try/catch blocks around all async configuration operations and validate loaded configuration before agent activation.
- **R-ASYNC-008** SHOULD: Consider implementing configuration caching to avoid redundant filesystem reads when multiple agents share configuration files.
- **R-ASYNC-009** MUST: Unit tests for agent initialization properly await configuration loading and verify async behavior.

### Verify

```bash
# Check for synchronous fs operations in agent code
grep -r "fs\.readFileSync" src/agents/ src/core/Subagents* --include="*.ts" | grep -v "test" || echo "No synchronous fs operations found in agent code"

# Count async load/read/apply functions
grep -r "async.*load\|async.*read\|async.*apply" src/agents/ src/core/Subagents* --include="*.ts" | wc -l

# Run tests for agent initialization and configuration loading
npm test -- --testPathPattern="(GeminiCliAgent|OpenCodeAgent|ZedAgent|SubagentsUtils|SubagentsProcessor)" --testNamePattern="async|configuration|loading"
```

**Accept when:**
- All agent configuration loading functions use async/await with fs/promises and no synchronous fs operations exist in agent initialization paths
- Unit tests for agent initialization properly await configuration loading and verify async behavior
- Code review confirms separation of I/O and parsing concerns with appropriate error handling for Promise rejections
- All async configuration functions include JSDoc annotations documenting file formats, error conditions, and return types
- Configuration loading completes before agents are marked as ready/initialized

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for agent initialization and configuration loading code paths. Violations detected by automated checks MUST block CI pipeline and code review merge.
</enforcement>