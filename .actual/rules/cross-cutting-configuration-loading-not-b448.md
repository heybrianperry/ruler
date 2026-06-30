# Standardize Asynchronous Configuration Loading Pattern for Agent Initialization: Configuration Loading Not

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading utilities within the codebase, including agent class implementations (GeminiCliAgent, OpenCodeAgent, ZedAgent, AbstractAgent), core utility modules (SubagentsUtils, SubagentsProcessor, FileSystemUtils), and functions that read agent settings, MCP configurations, and YAML/TOML subagent definitions.

### Rules

- **R-CONFIG-ASYNC-001** MUST NOT: Configuration loading MUST NOT use synchronous filesystem operations (fs.readFileSync) in agent initialization paths.

### Verify

```bash
# Check for synchronous fs operations in agent and configuration loading modules
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
- Configuration loading functions return parsed objects rather than raw strings, encapsulating both I/O and parsing concerns
- JSDoc annotations document expected file formats, error conditions, and return types for async configuration functions

<enforcement>
Claude Code MUST NOT skip or defer verification. CI build fails if synchronous filesystem operations are detected in agent initialization code. Code review blocks merge if async configuration loading lacks proper error handling. Static analysis warnings for missing await on configuration loading functions are escalated to errors.
</enforcement>