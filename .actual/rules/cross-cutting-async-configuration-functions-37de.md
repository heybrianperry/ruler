# Standardize Asynchronous Configuration Loading Pattern for Agent Initialization: Async Configuration Functions

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading utilities within the codebase, including agent class implementations (GeminiCliAgent, OpenCodeAgent, ZedAgent, AbstractAgent), core utility modules (SubagentsUtils, SubagentsProcessor, FileSystemUtils), and functions that read agent settings, MCP configurations, and YAML/TOML subagent definitions.

### Rules

- **R-ASYNC-CONFIG-001** SHOULD: Async configuration functions SHOULD use descriptive names indicating their asynchronous nature (e.g., loadSubagentFile, readGeminiSettings, discoverSubagents).
- **R-ASYNC-CONFIG-002** MUST: Use fs/promises import for all new configuration loading code; prefer `import { readFile } from "fs/promises"` over promisify(fs.readFile).
- **R-ASYNC-CONFIG-003** MUST: Structure configuration loading functions to return parsed objects rather than raw strings, encapsulating both I/O and parsing concerns.
- **R-ASYNC-CONFIG-004** MUST: Add JSDoc annotations to async configuration functions documenting expected file formats, error conditions, and return types.
- **R-ASYNC-CONFIG-005** MUST: In agent constructors or initialization methods, ensure all async configuration loading completes before marking agent as ready/initialized.
- **R-ASYNC-CONFIG-006** SHOULD: Consider implementing configuration caching to avoid redundant filesystem reads when multiple agents share configuration files.
- **R-ASYNC-CONFIG-007** MUST: Implement comprehensive error handling with try/catch blocks around all async configuration operations and validate loaded configuration before agent activation.
- **R-ASYNC-CONFIG-008** MUST: Implement file locking or atomic write operations (e.g., writeAgentsDirectoryAtomic) for configuration updates and use read-only access patterns during initialization.

### Verify

```bash
# Verify no synchronous fs operations in agent code
grep -r "fs\.readFileSync" src/agents/ src/core/Subagents* --include="*.ts" | grep -v "test" || echo "No synchronous fs operations found in agent code"

# Count async configuration functions
grep -r "async.*load\|async.*read\|async.*apply" src/agents/ src/core/Subagents* --include="*.ts" | wc -l

# Run tests for async configuration loading
npm test -- --testPathPattern="(GeminiCliAgent|OpenCodeAgent|ZedAgent|SubagentsUtils|SubagentsProcessor)" --testNamePattern="async|configuration|loading"
```

**Accept when:**
- All agent configuration loading functions use async/await with fs/promises and no synchronous fs operations exist in agent initialization paths
- Unit tests for agent initialization properly await configuration loading and verify async behavior
- Code review confirms separation of I/O and parsing concerns with appropriate error handling for Promise rejections
- All async configuration functions include JSDoc annotations documenting file formats, error conditions, and return types
- Configuration loading in agent constructors completes before agent is marked as ready/initialized

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if synchronous filesystem operations are detected in agent initialization code. Code review MUST block merge if async configuration loading lacks proper error handling. Static analysis warnings for missing await on configuration loading functions MUST be escalated to errors.
</enforcement>