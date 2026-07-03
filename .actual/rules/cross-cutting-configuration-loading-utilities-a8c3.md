# Standardize Asynchronous Configuration Loading Pattern for Agent Initialization: Configuration Loading Utilities

These rules are ALWAYS ACTIVE for all agent class implementations, core utility modules for configuration loading, and functions that read agent settings, MCP configurations, and YAML/TOML subagent definitions during initialization and setup methods in agent lifecycle.

### Rules

- **R-CONFIG-001** SHOULD: Configuration loading utilities SHOULD separate filesystem I/O concerns from parsing logic to enable independent testing.

### Verify

```bash
# Verify no synchronous fs operations in agent initialization code
grep -r "fs\.readFileSync" src/agents/ src/core/Subagents* --include="*.ts" | grep -v "test" || echo "No synchronous fs operations found in agent code"

# Count async configuration loading functions
grep -r "async.*load\|async.*read\|async.*apply" src/agents/ src/core/Subagents* --include="*.ts" | wc -l

# Run tests for async configuration loading
npm test -- --testPathPattern="(GeminiCliAgent|OpenCodeAgent|ZedAgent|SubagentsUtils|SubagentsProcessor)" --testNamePattern="async|configuration|loading"
```

**Accept when:**
- All agent configuration loading functions use async/await with fs/promises and no synchronous fs operations exist in agent initialization paths
- Unit tests for agent initialization properly await configuration loading and verify async behavior
- Code review confirms separation of I/O and parsing concerns with appropriate error handling for Promise rejections

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration loading functions MUST use async/await patterns with fs/promises. Synchronous filesystem operations in agent initialization code are violations that block CI builds.
</enforcement>