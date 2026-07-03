# Standardize Asynchronous Configuration Loading Pattern for Agent Initialization: Configuration Parsing Operations

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading utilities within the codebase, including agent class implementations (GeminiCliAgent, OpenCodeAgent, ZedAgent, AbstractAgent), core utility modules (SubagentsUtils, SubagentsProcessor, FileSystemUtils), and functions that read agent settings, MCP configurations, and YAML/TOML subagent definitions.

### Rules

- **R-CONFIG-001** MUST: Configuration parsing operations (JSON.parse, js-yaml, @iarna/toml) MUST occur after awaiting filesystem read operations.
- **R-CONFIG-002** MUST: Use fs/promises import for all new configuration loading code; prefer `import { readFile } from "fs/promises"` over promisify(fs.readFile).
- **R-CONFIG-003** MUST: Structure configuration loading functions to return parsed objects rather than raw strings, encapsulating both I/O and parsing concerns.
- **R-CONFIG-004** MUST: In agent constructors or initialization methods, ensure all async configuration loading completes before marking agent as ready/initialized.
- **R-CONFIG-005** SHOULD: Add JSDoc annotations to async configuration functions documenting expected file formats, error conditions, and return types.
- **R-CONFIG-006** SHOULD: Consider implementing configuration caching to avoid redundant filesystem reads when multiple agents share configuration files.
- **R-CONFIG-007** SHOULD: Implement comprehensive error handling with try/catch blocks around all async configuration operations and validate loaded configuration before agent activation.
- **R-CONFIG-008** SHOULD: Implement file locking or atomic write operations (e.g., writeAgentsDirectoryAtomic) for configuration updates and use read-only access patterns during initialization.

### Verify

```bash
# Verify no synchronous fs operations in agent code
grep -r "fs\.readFileSync" src/agents/ src/core/Subagents* --include="*.ts" | grep -v "test" || echo "No synchronous fs operations found in agent code"

# Count async configuration loading functions
grep -r "async.*load\|async.*read\|async.*apply" src/agents/ src/core/Subagents* --include="*.ts" | wc -l

# Run tests for agent initialization and configuration loading
npm test -- --testPathPattern="(GeminiCliAgent|OpenCodeAgent|ZedAgent|SubagentsUtils|SubagentsProcessor)" --testNamePattern="async|configuration|loading"
```

**Accept when:**
- All agent configuration loading functions use async/await with fs/promises and no synchronous fs operations exist in agent initialization paths
- Unit tests for agent initialization properly await configuration loading and verify async behavior
- Code review confirms separation of I/O and parsing concerns with appropriate error handling for Promise rejections
- Configuration parsing operations consistently follow filesystem read operations in the call sequence

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration loading code MUST comply with R-CONFIG-001 through R-CONFIG-008. Violations in agent initialization paths MUST be caught by CI pipeline linting rules and unit test coverage requirements.
</enforcement>